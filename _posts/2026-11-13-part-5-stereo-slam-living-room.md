---
title: "[build my robot] Part 5: Mapping My Living Room with Stereo Visual SLAM"
part: 5
excerpt: "A small Python stereo SLAM pipeline for the XLeRobot's fisheye camera: visual odometry, an occupancy grid, a TSDF mesh and loop closure, plus a comparison of SGBM and RAFT-Stereo depth."
date: 2026-11-13
categories:
  - xlerobot
tags:
  - robotics
  - build-log
  - computer-vision
  - stereo-vision
  - slam
toc: true
toc_sticky: true
---

{% include series-nav.html %}

{% include figure image_path="/assets/images/slam_mesh_comparison.jpg" alt="Six renderings of a living-room mesh in two rows, from the start position and looking at a sofa and a book display. Left column SGBM: bumpy floor, missing kitchen cabinet, blobby sofa. Middle and right columns RAFT-Stereo realtime and Middlebury: flat floor, a solid white cabinet, a sofa with visible back, seat, cushions and a teal blanket" caption="The same 300 frames fused into a mesh, with three depth sources. Left: SGBM. Middle: RAFT-Stereo realtime. Right: RAFT-Stereo Middlebury." %}

With the [stereo camera calibrated]({% post_url 2026-11-09-part-3-calibrate-stereo-fisheye-camera %}), the
next step was to let the robot map a room. I wanted three things:

- the camera's **trajectory** (visual odometry),
- a 2D **occupancy grid** the robot could later plan paths on,
- a dense 3D **mesh** of the room, mostly because it's fun to look at.

I'm working through [*14 Lectures on Visual SLAM*](https://github.com/gaoxiang12/slambook-en),
so instead of installing a big SLAM system I built a small Python pipeline from the book's
chapters. Everything runs on my MacBook Air. The code is in
[`slam/`](https://github.com/lhphanto/build_xlerobot/tree/main/slam).

## Why not ORB-SLAM3 or RTAB-Map?

I looked at the usual stereo SLAM systems first: ORB-SLAM3, VINS-Fusion, Basalt, RTAB-Map,
and newer learned ones like DROID-SLAM and MASt3R-SLAM. Two reasons pushed me to write my own:

- **Installation.** ORB-SLAM3 needs Pangolin and an OpenCV build from source. Homebrew's
  RTAB-Map pulled in about 125 packages and 6–8 GB, which my nearly full disk couldn't take.
- **Learning.** A pipeline made of pieces I'd already ported from the book (ORB matching, PnP,
  bundle adjustment) shows what each stage contributes, and I can swap any stage out.

The robot only drives around one living room, so a small, simple system is enough.

## Recording

`record_stereo.py` records the side-by-side 3840 × 1080 stream while I drive the robot with
the keyboard teleop script. The camera is mounted upside down, so frames are rotated 180°
to match the calibration.

### The camera was only giving me 1 frame per second

My first recordings tracked badly, and the reason turned out to be the recorder, not the
tracking. Of the 900 frames in a 90-second video, only 90 were different. ffmpeg's
`avfoundation` input on macOS delivered about **1 real fps** at 3840 × 1080 and repeated each
frame 10 times to fill the requested rate. With a full second between real frames, the robot
moved 15 cm or turned 25° from one image to the next.

The camera itself advertises 30–60 fps at that resolution. Reading it with OpenCV's
`VideoCapture` (also AVFoundation underneath) gave a real **30 fps**. The recorder now
captures with OpenCV, keeps every third frame for 10 fps, and pipes the frames to ffmpeg only
for encoding.

### Camera height and tilt

The camera sits about 1.2 m above the floor. Pointing nearly level, it couldn't see the
floor closer than about 1.8 m, so the area around the robot stayed unmapped. I tilted it down
a bit more; the pipeline now measures it at about 9° below level. The tilt doesn't affect the
stereo calibration, which only describes the two lenses relative to each other.

## The pipeline

The script is
[`stereo_vo_map.py`](https://github.com/lhphanto/build_xlerobot/blob/main/slam/stereo_vo_map.py),
written as notebook-style `# %%` cells. It makes three passes over the video.

### Pass 1: visual odometry

For each frame:

1. **Rectify** both halves with the fisheye calibration, at half resolution (960 × 540).
2. **Depth** from stereo matching, Z = f · b / d, as in the
   [previous post]({% post_url 2026-11-09-part-3-calibrate-stereo-fisheye-camera %}).
3. **ORB features**, matched against the last **keyframe**.
4. **Pose by 3D–2D matching:** the keyframe's features have 3D positions from its depth map,
   so `solvePnPRansac` gives the camera pose. A Huber-weighted Gauss–Newton step on SE(3) then
   refines it (slambook Section 6.7).
5. **New keyframe** when the inlier count drops below 120, or the camera has moved 0.2 m or
   turned 15° since the last one.

Tracking against a keyframe instead of the previous frame keeps errors from piling up while
the robot is standing still.

### Pass 2: a level world frame and the occupancy grid

Odometry runs in the first camera's frame, which is tilted. Since the robot drives on a flat
floor, the camera path lies in a plane parallel to the floor. That plane's normal gives
"up". I set the camera height to my tape measurement (1.20 m).

I also **snap every pose onto that plane**: fixed height, fixed tilt and roll, with only x, y
and heading free. Before this, odometry noise moved the height by ±2–4 cm and the tilt by a
few degrees. At a few meters' range, that was enough to lift the floor into the obstacle
height band.

The occupancy grid has 5 cm cells and stores log-odds (the log of the odds that a cell is
occupied):

- **Floor points**, within a tolerance that grows with distance, mark their cell and the cells
  along the ray from the camera as **free**.
- **Points above the floor and below 1.4 m** mark their cell **occupied** and the ray before it
  free.
- A cell needs at least two keyframes' worth of evidence before it shows as free or occupied.

{% include figure image_path="/assets/images/slam_occupancy_grid.jpg" alt="Top-down occupancy grid in meters: a white free area around a red camera path that starts at the origin and loops forward about 3 meters; black occupied bands on the right are the sofa and book display, and black blobs on the left are the kitchen counter and chairs" caption="The occupancy grid. The sofa and book display are the black band on the right, the kitchen counter and chairs on the left." %}

### Pass 3: a dense mesh by TSDF fusion

A point cloud made by stacking depth maps keeps all their noise. Chapter 12 of the book
covers the better approach, a **TSDF** (truncated signed distance function):

- Each 2 cm voxel stores its distance to the nearest surface along the camera ray, positive
  in front of the surface and negative behind it.
- Every frame updates the voxels it sees with a running weighted average. Noise from
  individual frames averages out.
- The surface is where the distance crosses zero. Marching cubes turns that into a triangle
  mesh.

I used Open3D's `VoxelBlockGrid`, which only allocates voxels near surfaces. Depth beyond
2.5 m and pixels on depth edges are dropped first, and a surface has to be seen in at least
3 frames to stay in the mesh.

## SGBM vs. RAFT-Stereo

My first meshes had a rough floor and shapeless furniture. The cause was mostly the depth.
So I compared four ways of computing it on the same recording (300 frames, every third):

- **SGBM, half resolution:** OpenCV's semi-global block matching at 960 × 540. This is what
  the pipeline used so far.
- **SGBM, full resolution:** the same at 1920 × 1080.
- **[RAFT-Stereo](https://github.com/princeton-vl/RAFT-Stereo), realtime weights:** a neural
  network for stereo matching, smaller and faster variant.
- **RAFT-Stereo, Middlebury weights:** the full model, fine-tuned on indoor scenes.

Full-resolution SGBM alone ran my laptop out of memory, and the network is heavier still. So
I wrote a batch script,
[`stereo_depth_batch.py`](https://github.com/lhphanto/build_xlerobot/blob/main/slam/stereo_depth_batch.py),
ran it on a remote machine with an NVIDIA L40 GPU, and copied the disparity maps back. The
script uses exactly the same rectification as the pipeline, which can now load those maps
instead of running SGBM.

{% include figure image_path="/assets/images/slam_depth_comparison.jpg" alt="Grid of three frames from the living room, each shown as the camera image followed by four depth maps. The two SGBM columns are speckled with many black holes on walls and floor; the two RAFT-Stereo columns are smooth and nearly complete with sharp chair and sofa outlines; only windows and the black TV stay black" caption="Three frames, with depth from SGBM (half and full resolution) and RAFT-Stereo (realtime and Middlebury). Red is near, blue is far, black is no depth." %}

There's no ground-truth depth for my living room, so I used the floor as the reference: it's
flat, and I know the camera's height from a tape measure. For each frame I fitted a plane to
the floor, then measured how much the floor points scatter around it and how far below the
camera it is.

| Method | Pixels with depth | Floor scatter at 0.5–1.5 m | at 1.5–2.5 m | at 2.5–3.5 m | Camera height (tape: 1.20 m) | Time per frame |
|---|---|---|---|---|---|---|
| SGBM, half | 64% | 4.8 cm | 5.2 cm | 6.1 cm | 1.233 m | 0.14 s (CPU) |
| SGBM, full | 54% | 5.7 cm | 6.1 cm | 7.0 cm | 1.232 m | 0.48 s (CPU) |
| RAFT realtime | 89% | 3.0 cm | 4.0 cm | 5.6 cm | 1.223 m | 0.07 s (L40) |
| **RAFT Middlebury** | **88%** | **2.4 cm** | **3.5 cm** | **5.0 cm** | 1.235 m | 0.18 s (L40) |

Floor scatter is a robust spread (1.4826 × median absolute deviation), so toys on the floor
don't dominate it. The times are for the whole batch script per frame: decoding the video,
rectifying, matching and saving. The matching alone is a fraction of that; SGBM at half size
takes about 0.03 s.

What I take from this:

- **RAFT-Stereo halves the floor noise up close** and gives depth for about 25 percentage
  points more of the image. The difference is clearest on plain surfaces like walls, cabinet
  doors and the sofa fabric. SGBM needs texture to match; the network fills those areas in
  from context.
- **Far away, every method is limited by the 6 cm baseline.** At 3 m the two images differ by
  only about 6 pixels, so a quarter-pixel matching error is already 12 cm of depth. All four
  methods end up at 5–7 cm of scatter there. That's why the TSDF only uses depth up to 2.5 m.
- **Full-resolution SGBM is worse than half resolution, and 3–4 times slower.** SGBM compares
  5 × 5 pixel patches. At full resolution a patch covers a quarter of the scene area and holds
  less texture, so more matches fail.
- **The depth scale is right.** All four methods put the floor 1.22–1.24 m below the camera,
  within about 3 cm of my tape measurement. That rules out a calibration error.
- **The realtime model gets most of the benefit at 2.6× the speed.** For offline mapping I use
  the Middlebury model, since 900 frames take under 3 minutes on the L40. For live mapping
  the realtime model is the candidate.

The mesh at the top of this post shows the result. With RAFT depth, the floor is flat, the
white kitchen cabinet is a solid block instead of a few scraps, the wall behind the sofa is
filled in, and the sofa has a recognizable back, seat, cushions and blanket.

## Loop closure

Odometry chains each keyframe to the previous one, so small errors add up (drift). When the
robot sees a place again, that's a second, independent measurement which can pull the
trajectory back into shape. This is loop closure (chapters 10 and 11 of the book).

The robot doesn't need to return to the same pose. It needs to see the same part of the
room again from a similar enough angle that features still match. ORB copes with scale
changes of about 2× and with viewpoints up to roughly 30–45° apart.

My implementation:

1. **Candidates:** match each keyframe's ORB features against every keyframe at least 8
   keyframes older. ORB-SLAM uses a bag-of-words index for this step; with about 50 keyframes
   in one room, brute force takes 13 seconds.
2. **Geometric check:** run the same PnP + Huber refinement as in tracking. At least 80
   inliers turns the pair into a loop with a measured relative pose.
3. **Pose graph:** keyframe poses are the nodes. Odometry between consecutive keyframes and
   the loops are the edges. SciPy's `least_squares` with a Huber loss finds the poses that
   best agree with all edges, so a single wrong loop can't drag the whole map.
4. Every frame then moves along with its keyframe.

On this recording it found **29 loops** among 45 keyframes. They came from passing the
start area again and from a second pass along the sofa. Drift turned out to be small:
odometry disagreed with the loops by only 2–12 cm and 1–3°. After optimization the
keyframes moved by up to 14 cm, and the sofa's doubled edges from the two passes became
slightly sharper. The remaining blur comes from depth noise at 2–3 m, not from drift.

## Lessons learned

### 1. Check the real frame rate

**Problem:** ffmpeg's macOS camera input delivered 1 fps at full resolution and quietly
duplicated frames to the requested rate.

**Fix:** Count distinct frames in a recording. Capture with OpenCV instead, which got a real
30 fps from the same camera.

### 2. Use what you know about the robot

**Problem:** Small errors in the camera's height and tilt turned the far floor into
obstacles.

**Fix:** The robot drives on a flat floor with a fixed camera mount. Snapping the poses to a
plane at the measured height removes those errors entirely.

### 3. Depth quality decides map quality

**Problem:** Dense meshes from SGBM had bumpy floors and missing walls.

**Fix:** A learned stereo model (RAFT-Stereo) halved the floor noise and filled the gaps.
Running it at full resolution or adding more frames would have helped much less.

### 4. Know where the baseline runs out

**Problem:** With lenses 6 cm apart, depth error grows with distance squared: about 1.4 cm at
1 m, 5.5 cm at 2 m and 12.5 cm at 3 m.

**Fix:** Cut depth off at 2.5 m for the mesh, and drive closer to anything I want to see in
detail.

### 5. Don't run big experiments on a laptop

**Problem:** Full-resolution SGBM ran my MacBook Air out of memory, and RAFT-Stereo needs a
GPU to be practical.

**Fix:** Move the heavy depth computation to a GPU machine with a standalone batch script,
and keep the laptop for tracking and fusion.

## Next

- Drive a bigger loop around the room and end at the start, to test loop closure on real
  drift.
- Log wheel odometry during teleop and fuse it with the visual odometry, the way VINS fuses
  an IMU.
- Use the occupancy grid for path planning, so the robot can drive around the room on its
  own.
