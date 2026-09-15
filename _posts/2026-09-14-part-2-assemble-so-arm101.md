---
title: "Part 2: Assemble the SO-ARM101"
excerpt: "Assembling the SO-ARM101 follower arm and setting up its STS3215 servos."
date: 2026-09-14
categories:
  - xlerobot
tags:
  - robotics
  - build-log
  - so-arm101
toc: true
toc_sticky: true
---

{% include series-nav.html %}

{% include figure image_path="/assets/images/so_arm101_follower.jpg" alt="An assembled SO-ARM101 follower arm lying on a wooden table, with its servo cables routed along the links and the control board at the base" caption="The assembled SO-ARM101 follower arm." %}

<!-- TODO: intro - where the build is at after printing, and what this post covers. -->

## Setting up the servos

For calibrating the servos, I first tried
[feetech-servo-tool](https://github.com/dgmz/feetech-servo-tool), but it didn't work
very well. In particular, it can't set **Maximum Acceleration**.

So I ended up writing my own script, with the help of Claude Code of course :-).
It's in [build_xlerobot](https://github.com/lhphanto/build_xlerobot).

<!-- TODO: notes from the servo setup session, to write up in your own words:
     - `lerobot-setup-motors` only assigns each motor its ID and baud rate (1 Mbps),
       one motor at a time, starting from the gripper. It never re-centres a servo;
       `lerobot-calibrate` does that later by writing Homing_Offset.
     - Connect exactly one servo during setup: new STS3215s all ship as ID 1, and two
       servos with the same ID garble each other's replies.
     - servo_regs.py (build_xlerobot repo) for scanning, dumping registers, health
       checks, re-centring (Torque_Enable = 128) and recording position limits. -->

## Assembly

I mainly followed the
[LeRobot SO-ARM101 Robotic Arm – Assembly and Setup Guide](https://www.youtube.com/watch?v=70GuJf2jbYk):

{% include video id="70GuJf2jbYk" provider="youtube" %}

For a closer look at how each servo is inserted and which way it should face, Waveshare's
[SO-ARM100/101 assembly tutorial](https://www.youtube.com/watch?v=rVP1XQ0PeM4) is a
useful second reference.

## Lessons learned

### 1. Power the servos with 12 V

<!-- TODO: Problem / Fix, from the power-supply notes:
     - STS3215 C018 is a 12 V servo; Max_Voltage_Limit is 14.0 V.
     - 100 W USB-C charger: servo went completely silent on the bus (likely
       overvoltage protection).
     - 15 W USB-C charger: servo answered but only measured 5.1 V - no torque.
     - Anker SOLIX 140 W + USB-C to 12 V cable: measured 12.2 V and worked. -->

### 2. Use the cable that came with the servos

<!-- TODO: Problem / Fix: the homemade daisy-chain cable broke communication with the
     whole bus; the cable that came with the STS3215 worked. Check pin order against
     the stock cable before blaming a servo. -->

### 3. Centre each joint before setting `Torque_Enable` to 128

**Problem:** Writing 128 to `Torque_Enable` makes the servo treat its current position
as its new centre. If the joint isn't in the middle of its range at that moment, the
centre ends up in the wrong place.

**Fix:** During calibration, move each joint to the middle of its range first, as shown
in the [assembly guide](https://www.youtube.com/watch?v=70GuJf2jbYk), then set
`Torque_Enable` to 128.

### 4. Widen tight screw holes with a drill

**Problem:** A few holes were slightly too tight for the screws.

**Fix:** I opened them up a little with a drill.

## Next

<!-- TODO -->
