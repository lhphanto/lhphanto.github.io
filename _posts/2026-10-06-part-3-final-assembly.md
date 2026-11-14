---
title: "[build my robot] Part 3: Final Assembly of the Robot Body"
part: 3
excerpt: "Putting XLeRobot's body together on the IKEA cart, and swapping the stock omni-wheel base for a four-wheel mecanum drive."
date: 2026-10-06
categories:
  - xlerobot
tags:
  - robotics
  - build-log
  - mecanum
toc: true
toc_sticky: true
---

{% include series-nav.html %}

<!-- TODO: photo of the assembled robot. Drop it in assets/images/ and use:
{% include figure image_path="/assets/images/xlerobot_assembled.jpg" alt="..." caption="..." %}
-->

With both SO-ARM101 arms built in [Part 2]({% post_url 2026-09-14-part-2-assemble-so-arm101 %}),
this part puts the rest of the body together: the IKEA RÅSKOG cart, the wheel base, the
arm base and head, and the wiring. The one change from the standard build is the base: I
used a four-wheel **mecanum** drive instead of the default omni wheels.

## Assembly

I followed the [XLeRobot assembly guide](https://xlerobot.readthedocs.io/en/latest/hardware/getting_started/assemble.html)
and its official video from WowRobo, which matches the 0.3 version I printed in
[Part 1]({% post_url 2026-09-07-part-1-3d-printing %}):

{% include video id="upB1CEFeOlk" provider="youtube" %}

Greg's Tech's [We assembled an XLeRobot. Here are all our mistakes.](https://www.youtube.com/watch?v=Ezy3EepLSm4)
is worth watching before you start, too.

The guide estimates 1–2 hours if the arms are already built. In order:

1. **IKEA cart:** assemble the RÅSKOG cart.
2. **Wheel base:** mount the wheels, wire the motors, and attach the top plate. Check that
   all four cart wheels still touch the ground.
3. **Base into the cart:** flip the cart, install the base, and cut out the central "X" of
   the metal mesh so cables can pass through.
4. **Arm base and head:** assemble the top base (easier upside down) and the head.
5. **Wiring:** route all the cables before clamping the top base into the cart.
6. **Battery:** put it on the middle or lower shelf to keep the center of mass low.
7. **Final assembly:** fit the cart edge into the case socket and clamp the arms to the top
   corners.

A few warnings from the guide that are easy to miss:

- **Connect the power cables last**, to protect the motor control boards, and unplug power
  before plugging or unplugging anything else.
- **Check polarity on any 5264 cable you extend yourself.** Reversed wires cause errors.
  This is the same trap as the homemade cable in Part 2.
- **Lift the robot; don't push it around like a cart.** Pushing back-drives the wheel motors
  and can damage their gears. Assembled, it weighs about 12 kg.

<!-- TODO: anything that didn't fit or that you'd do differently during assembly. -->

## Mecanum wheels instead of omni wheels

The standard base uses omni wheels. I followed XLeRobot's
[mecanum wheel assembly](https://xlerobot.readthedocs.io/en/latest/hardware/getting_started/assemble_mecanum.html)
instead. The rest of the build stays the same; only the wheels and their couplings change.

**Why mecanum:**

- **Holonomic motion.** The base can strafe sideways and turn in place.
- **More torque and traction**, because four motors drive the base.
- **The cost:** two extra servos and more wiring.

That page is still marked as under construction. For the printed parts, it points to the
dual-wheel build's files (`XLeRobot_0_4_0_extra.stl`) plus a wheel-to-motor coupling,
`mecanum_wheel_motor_coupling.stl`, with its CAD source in `hardware/mecanum/`.

### The wheels

The guide doesn't name a specific wheel. One option I found is the
[100mm Mecanum Wheel Set](https://www.robotshop.com/products/100mm-mecanum-wheel-set)
from Nexus Robot on RobotShop, about $149 for four:

- 2 left-hand and 2 right-hand wheels, 100 mm diameter
- 9 rollers per wheel; PP+PE rollers on an aluminum alloy body

The 100 mm size matches XLeRobot's mecanum software, which assumes a 0.05 m wheel radius.
The listing doesn't give the hub or bore size, so check that the hub fits the printed
coupling before you order.

<!-- TODO: confirm which wheels you actually used, and how they attached to the coupling. -->

### X layout

Mecanum wheels come in mirrored pairs, so a left-hand wheel can't go on a right-hand
corner. Mount them in an **X**: seen from above, the rollers on the four corners should
line up like an X through the center of the base. XLeRobot's software assumes this
layout. The other arrangement (an O, or diamond) won't match the kinematics unless you
change the model, and the usual symptom is that forward motion works but strafing goes
diagonal.

### Wiring and motor IDs

The four wheel servos share a bus with the right arm: IDs 1–6 are the arm, and 7–10 are
the wheels. The guide's order is 7 = front-left, 8 = front-right, 9 = rear-left,
10 = rear-right.

My wiring ended up different: **7 = rear-left, 8 = front-left, 9 = front-right,
10 = rear-right.** See the first lesson below for why that matters.

To confirm all four wheel servos answer before closing up the base, scan the right arm's
bus with my [`servo_regs.py`](https://github.com/lhphanto/build_xlerobot):

```bash
python servo_regs.py scan --no-dump --baud 1000000
```

### Driving it from the keyboard

XLeRobot's mecanum setup asks you to copy its `xlerobot_mecanum` module into lerobot and
edit the keyboard teleop example. I wrote a standalone script instead,
[`teleop_keyboard_base.py`](https://github.com/lhphanto/build_xlerobot), which talks to
the four wheel servos directly. It uses XLeRobot's kinematics and key bindings, takes the
wheel IDs as a flag, and has a dry-run mode that prints the wheel commands without moving
anything:

```bash
python teleop_keyboard_base.py --port /dev/cu.usbmodemXXXX --dry-run
python teleop_keyboard_base.py --port /dev/cu.usbmodemXXXX
```

Keys: `i`/`k` forward and back, `j`/`l` strafe, `u`/`o` rotate, `n`/`m` speed, space to
stop, `b` to quit. The base stops on its own when you let go of the keys.

## Lessons learned

### 1. Match the wheel IDs to the kinematics order

**Problem:** The software assumes a fixed wheel order: front-left, front-right, rear-left,
rear-right. With my wiring, the command meant for the front-left wheel would have gone to
the rear-left one. Forward motion still works by luck, because both left wheels spin the
same way. Strafing and turning don't, because they need the front and rear wheels to
differ.

**Fix:** Tell the software which ID is on which wheel instead of rewiring. My script takes
the order as a flag (`--wheel-ids 8,9,7,10` for my robot). In XLeRobot's own code, edit the
motor IDs in `xlerobot_mecanum/xlerobot.py`. Then lift the base so the wheels spin freely
and check each direction before driving it on the floor.

### 2. Make sure the DC power cable delivers 12 V

**Problem:** The STS3215 servos are 12 V parts, and their `Max_Voltage_Limit` is 14.0 V.
I power the control boards from USB-C through a USB-C to DC cable, and what the cable
outputs depends on what the charger and cable negotiate:

- **100 W charger:** the servos went completely silent on the bus. They powered up but
  never answered, because a voltage above 14 V trips their overvoltage protection.
- **15 W charger:** the servos answered, but measured only 5.1 V, which isn't enough to
  drive the motors.
- **USB-C to 12 V DC cable:** measured 12.2 V, and everything worked.

From the computer, a servo stuck in overvoltage protection looks exactly like a dead one.

**Fix:**

- Use a USB-C to DC cable that outputs a fixed **12 V**, such as
  [these](https://www.amazon.com/clp/B0CDBWHXDZ).
- Prefer one rated for **at least 3 A**. Several servos share each cable, so it needs
  current headroom as well as the right voltage.
- Check the voltage before trusting the setup. Measure the DC plug with a multimeter, or
  run `servo_regs.py test` on any servo, which reports the voltage the servo actually sees.

### 3. Watch for friction between the wheels and their bearings

**Problem:** My wheels rubbed against the wheel bearings. The gap between them was too
tight, so the wheels had extra friction instead of spinning freely.

<figure>
  <video controls playsinline preload="metadata"
         poster="{{ '/assets/videos/wheel_friction_before_vaseline.jpg' | relative_url }}"
         style="max-width: 320px; width: 100%;">
    <source src="{{ '/assets/videos/wheel_friction_before_vaseline.mp4' | relative_url }}" type="video/mp4">
  </video>
  <figcaption>Before the Vaseline: the base drives unevenly, and you can hear the wheels rubbing.</figcaption>
</figure>

**Fix:** I put some Vaseline on the contact area, which was enough. A thicker bearing
should also help, because it adds clearance between the wheel and the bearing. The
mecanum guide specifies 30 × 37 × 4 mm bearings, so try a thicker one with the same inner
and outer diameter.

<!-- TODO: add your own lessons in the same Problem / Fix format. -->

## Next

[Part 4]({% post_url 2026-11-09-part-4-calibrate-stereo-fisheye-camera %}): calibrating the
stereo fisheye head camera.
