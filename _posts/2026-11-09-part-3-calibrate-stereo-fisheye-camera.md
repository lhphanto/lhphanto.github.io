---
title: "Part 3: Calibrate the Stereo Fisheye Camera"
excerpt: "Calibrating XLeRobot's stereo fisheye camera with Zhang's method, then turning image pairs into depth maps and 3D point clouds."
date: 2026-11-09
categories:
  - xlerobot
tags:
  - robotics
  - build-log
  - computer-vision
  - stereo-vision
toc: true
toc_sticky: true
---

{% include series-nav.html %}

{% include figure image_path="/assets/images/stereo_point_cloud.jpg" alt="A colored 3D point cloud of a bedroom reconstructed from one stereo image pair: a wall with hanging light-bulb decals, windows above, and a table covered in LEGO in the foreground" caption="A 3D point cloud computed from a single image pair from the robot's stereo camera." %}

XLeRobot's head camera is a cheap
[stereo fisheye USB camera](https://www.amazon.com/dp/B0CBLZJZBT): two wide-angle lenses
that deliver one 3840 × 1080 side-by-side frame, which is 1920 × 1080 per lens. To get
depth out of it, I first needed to know each lens's intrinsics and distortion, plus
exactly where the two lenses sit relative to each other. Nothing like that is stored in the
images, so I calibrated the camera myself.

## Calibration with Zhang's method

I used [Zhang Zhengyou's method](https://www.microsoft.com/en-us/research/publication/a-flexible-new-technique-for-camera-calibration/),
the standard approach and the one OpenCV implements. You show the camera a flat pattern
with known geometry from many angles. Each view gives a homography between the board and
the image, which constrains the intrinsics. A final nonlinear least-squares fit refines the
intrinsics, the lens distortion and each view's pose together.

### The board

I printed a letter-size checkerboard
([PDF](/assets/files/checkerboard_9x6_25.4mm_letter.pdf))
with 10 × 7 squares, giving **9 × 6 inner corners**, and **25.4 mm** squares. I taped it to
the lid of my MacBook Pro, which made a flat, rigid board for free.

Two details matter here:

- **Print at 100% / "Actual size".** The page has a 100 mm bar for checking the scale with a
  ruler. The square size sets the scale of everything downstream: if it's 2% off, the
  baseline between the lenses and every depth value are 2% off too.
- **Keep the board flat.** Zhang's method assumes every corner lies on one plane (Z = 0).
  A curled sheet of paper breaks that assumption.

### Capturing the views

Instead of taking individual snapshots, I recorded a video while moving the board around,
then extracted still frames from it. I ended up with **24 usable views** for each lens.

{% include figure image_path="/assets/images/stereo_calib_board.jpg" alt="Side-by-side frame from the stereo camera showing the checkerboard, taped to a laptop lid, held tilted in front of the camera in both the left and right halves" caption="One calibration view: the checkerboard on the MacBook lid, seen by both lenses at once." %}

In theory three views are enough to solve for the intrinsics. In practice you want 15 or
more, because:

- corner detection is noisy, and more views average the noise out;
- fisheye distortion is strongest near the image edges, so the board has to visit the edges
  and corners of the frame;
- tilting the board in different directions is what separates focal length from distance.

### Running the calibration

The script is
[`calibrate_stereo.py`](https://github.com/lhphanto/build_xlerobot/blob/main/calibrate_stereo.py).
It does two things:

1. **Calibrates each lens on its own** with OpenCV's fisheye model (`cv2.fisheye`, four
   distortion coefficients k1–k4). The standard pinhole distortion model doesn't fit lenses
   this wide well.
2. **Calibrates the pair** with `cv2.fisheye.stereoCalibrate`. Both lenses see the same
   board at the same moment, so the board's pose in each lens gives the rotation R and
   translation T between them. Each lens's intrinsics stay fixed in this step.

```sh
python calibrate_stereo.py --dir samples/calibration --fisheye --max-view-error 2
```

`--max-view-error 2` drops any view whose reprojection error is above 2 px. One blurry
frame was dropped this way.

### Results

| | Left lens | Right lens |
|---|---|---|
| fx, fy (px) | 600.9, 604.4 | 602.1, 605.0 |
| cx, cy (px) | 967.5, 522.6 | 980.9, 555.3 |
| RMS reprojection error | 0.70 px | 0.70 px |

Between the lenses:

- **Baseline: 60.5 mm**, almost purely sideways (0.08 mm vertical, 0.58 mm forward).
- **Relative rotation: 0.46°.**
- Stereo RMS reprojection error: 0.79 px.

These pass the sanity checks: the two lenses have nearly the same focal length, the offset
between them is nearly all horizontal, and the errors are under a pixel.

## From two images to depth

With the calibration done, the pipeline in the
[notebook](https://github.com/lhphanto/build_xlerobot/blob/main/Calibrate_two_fisheye_lenses.ipynb)
is based on the stereo example in [*14 Lectures on Visual SLAM*](https://github.com/gaoxiang12/slambook-en)
(Section 4.4.1), adapted for fisheye lenses. Here's a sample image pair:

{% include figure image_path="/assets/images/stereo_sample_pair.jpg" alt="Side-by-side frame of a bedroom from the stereo camera: a long window, light-bulb wall decals, a LEGO-covered table and a white chair, seen from two slightly different positions" caption="A sample pair: left lens on the left, right lens on the right." %}

### 1. Rectify

`cv2.fisheye.stereoRectify` and `cv2.fisheye.initUndistortRectifyMap` use the calibration to
remove the fisheye distortion and rotate both images, so that **the same point appears on
the same row in both images**. Stereo matching depends on that: it only has to search along
a row instead of over the whole image.

### 2. Disparity

OpenCV's semi-global block matching (`cv2.StereoSGBM`) finds, for each pixel in the left
image, how far its match has shifted in the right image. That shift is the disparity.
Closer objects shift more, so they're brighter:

{% include figure image_path="/assets/images/stereo_disparity.jpg" alt="Grayscale disparity map of the bedroom scene: the table in the foreground is bright, the wall behind it is darker grey, and featureless or overexposed areas are black" caption="SGBM disparity. Black pixels have no reliable match." %}

### 3. Depth

Depth follows from similar triangles: **Z = f · b / d**, where f is the focal length after
rectification, b the baseline and d the disparity.

{% include figure image_path="/assets/images/stereo_depth.jpg" alt="Depth map with a color scale from 0.25 to 2 meters: the LEGO table is purple at around 0.5 meters, the wall is orange at around 1.2 to 1.5 meters, and the far corner is yellow at about 2 meters" caption="Depth in meters, in the left camera's frame. The table is about 0.5 m away and the wall about 1.3 m." %}

The numbers look right for the room: the table is about half a meter from the camera and
the back wall a bit over a meter.

### 4. Point cloud

`cv2.reprojectImageTo3D` turns each pixel's disparity into an (X, Y, Z) point. I colored the
points from the left image and plotted them with Plotly. The result is the point cloud at the
top of this post. The wall decals, the Christmas tree and the LEGO on the table all land
where they should in 3D.

## Lessons learned

### 1. Use the fisheye model for wide-angle lenses

**Problem:** The standard pinhole distortion model can't describe distortion this strong,
especially near the edges.

**Fix:** Use `cv2.fisheye.calibrate` and `cv2.fisheye.stereoCalibrate`, and remember that
the resulting coefficients only work with other `cv2.fisheye` functions.

### 2. Cover the edges of the frame

**Problem:** My views covered roughly the central 70% of each image. Outside that area the
distortion model is extrapolated, and it gave an implausible field of view for one lens.

**Fix:** Trust the calibration in the covered area. For a better result, record more views
with the board near the edges and corners of the frame.

### 3. Hold each pose still when recording

**Problem:** At full resolution the camera delivers only about one frame per second, so
frames grabbed while the board is moving come out blurred.

**Fix:** Hold each board pose for about 4 seconds, and drop views with high reprojection
error.

### 4. Rectify before matching

**Problem:** Running SGBM on raw fisheye images produces mostly noise, because matching
points don't share a row.

**Fix:** Always undistort and rectify both images first.

## Next

Depth is still noisy on plain walls and overexposed windows, where there is little texture
to match. Next I want to use the depth and point cloud on the robot itself.
