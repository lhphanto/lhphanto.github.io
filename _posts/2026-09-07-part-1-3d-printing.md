---
title: "Part 1: 3D Printing the Parts"
excerpt: "Starting the XLeRobot build at the printer: why one elbow part needs tree supports, and why 1 kg of filament isn't enough."
date: 2026-09-07
categories:
  - xlerobot
tags:
  - robotics
  - build-log
  - 3d-printing
toc: true
toc_sticky: true
---

{% include series-nav.html %}

Starting my [XLeRobot](https://github.com/Vector-Wangel/XLeRobot) build: an open-source
dual-arm mobile robot built on LeKiwi, dual SO-101 arms, and an IKEA RÅSKOG cart.
Everything begins at the printer, so this post covers the print run and what I'd do
differently.

## Setup

- **Printer:** [Bambu Lab X2D](https://bambulab.com/en-us/x2d), a dual extruder with a
  nozzle dedicated to support material. That's more printer than this build needs; the
  guide notes a Bambu A1 (around $350) is enough.
- **Filament:** Bambu Lab PLA Matte, Charcoal, for the structural parts, and **TPU95A**
  for the soft gripper fingers.
- **Models:** XLeRobot version 0.3,
  [`XLeRobot_0_3_0.3mf`](https://github.com/Vector-Wangel/XLeRobot/blob/main/hardware/XLeRobot_0_3_0.3mf)
  from the project's `hardware/` folder. It's a `.3mf` file, so it opens directly in
  slicers like Bambu Studio.

The [XLeRobot 3D printing guide](https://xlerobot.readthedocs.io/en/latest/hardware/getting_started/3d.html)
recommends plain PLA (the demo units use PLA Matte Black), with PETG HF, PLA CF, or
Tough PLA as stronger alternatives.

## What gets printed

Per the guide:

- Two **SO-101 follower arms** (the leader arm is optional, for dual-arm teleoperation)
- The **arm base**, including a middle storage shell for boards and cable routing
- **Neck and head**, sized for SO-101 motors
- **Wheel base** components for the omni-wheels
- Optional protective shells for control boards and joints

The soft gripper fingers are printed in TPU95A; the original rigid SO-101 fingers are an
option if you'd rather stick to PLA. The guide puts total filament cost at $15–25.

## Lessons learned

### 1. Use tree supports on curved overhangs

**Problem:** One of the SO-ARM101 elbow parts failed partway up. Its overhanging tabs
had nothing underneath them, so the nozzle extruded into open air and the print turned
into "spaghetti", a nest of loose filament. The guide leaves supports up to you:

> Place, orient, and add supports yourself in the slicing software to ensure the best
> printing quality.

{% include figure image_path="/assets/images/wrist_without_support.jpg" alt="Printed arm part whose two overhanging tabs failed into loose strands of filament" caption="Without supports: nothing under the overhangs, so the nozzle extruded into open air." %}

**Fix:** Reprint the part with **tree supports**. Normal supports grow straight up from
the build plate as a solid block; tree supports branch in from the side and reach under
curved overhangs, where normal supports don't make contact.

{% include figure image_path="/assets/images/wrist_with_support.jpg" alt="The same part on the build plate, held up by branching tree supports under its overhangs" caption="With tree supports: branches reach under the overhangs from the build plate." %}

**Keep in mind:**

- **Preview every part in the slicer** before starting a multi-hour print, and check
  that every overhang has something under it.
- **Tree supports for curved overhangs, normal supports for flat ones.**
- **A dedicated support nozzle doesn't fix this.** The X2D's second nozzle makes
  supports easier to remove and cleaner where they touch the part, but whether an
  overhang gets supported depends only on the support type and the part's geometry.
- New to supports? [This video](https://youtu.be/89WspiTc5Z0) is a good primer on when
  and why to use them.

<!-- TODO: note the exact STL filename of this part. -->

### 2. Order all the filament up front

**Problem:** I started with a single 1 kg spool of PLA, and it ran out partway through
the part list. With no backup ordered, the build stopped for **four days** waiting on
delivery. The guide's $15–25 estimate reads like one spool, but the two arms, base,
storage shell, neck, head, and wheel base together need more.

**Fix:**

- **Order two 1 kg spools of PLA.** A spare spool costs about $20 and will get used
  eventually; four idle days cost more.
- **Order the TPU95A in the same order.** It's a small amount and comes late in the part
  list, so it's easy to forget, and forgetting it means a second delivery wait. PLA is
  too rigid for the soft gripper fingers.

### 3. Don't feed TPU95A through the AMS

<div class="notice--warning" markdown="1">
**Problem:** TPU95A doesn't work with the Bambu AMS. Soft TPU can
[get stuck in the printer](https://www.reddit.com/r/3Dprinting/s/Q4gBEKyso7).

**Fix:** Feed TPU without the AMS. On the X2D, follow Bambu Lab's
[TPU printing guide](https://wiki.bambulab.com/en/x2d/manual/tpu-printing-guide).
</div>

## Next

Assembling the SO-ARM101 and setting up its servos.
