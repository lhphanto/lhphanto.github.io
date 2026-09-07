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

Starting my [XLeRobot](https://github.com/Vector-Wangel/XLeRobot) build — an open-source
dual-arm mobile robot built on LeKiwi, dual SO-101 arms, and an IKEA RÅSKOG cart.

Everything begins at the printer, so this post covers the print run and the two things
I'd do differently — one that cost me a part, one that cost me four days.

## The setup

**Printer:** [Bambu Lab X2D](https://bambulab.com/en-us/x2d) — dual extruder, with an
auxiliary nozzle dedicated to support material.

**Filament:**
- Bambu Lab PLA Matte, Charcoal — for the structural parts
- **TPU95A** — required for the soft gripper fingers. PLA will not work here.

The [XLeRobot 3D printing guide](https://xlerobot.readthedocs.io/en/latest/hardware/getting_started/3d.html)
recommends plain PLA and notes the demo units were printed in PLA Matte Black. PETG HF,
PLA CF, or Tough PLA are listed as stronger alternatives if you want them. The X2D is
considerably more printer than this build needs — the docs point out a Bambu A1 at
around $350 is entirely sufficient.

One prep step worth not skipping: the docs suggest **drying PLA at 45 °C for 8 hours**
before printing if you're anywhere humid.

## What gets printed

Per the guide:

- Two **SO-101 follower arms** (the leader arm is optional, for dual-arm teleoperation)
- The **arm base**, including a middle storage shell for boards and cable routing
- **Neck and head**, sized for SO-101 motors
- **Wheel base** components for the omni-wheels
- Optional protective shells for control boards and joints

Soft gripper fingers need TPU95A rather than PLA — the original rigid SO-101 fingers are
available if you'd rather not switch materials.

Total filament cost lands around $15–25.

## The lesson: tree supports on the elbow

The docs are deliberately hands-off about supports:

> Place, orient, and add supports yourself in the slicing software to ensure the best
> printing quality.

That's reasonable — orientation depends on your printer and plate layout. But it means
the failure modes are yours to discover, and I found one.

One of the **elbow parts for the SO-ARM101** failed partway up with default supports.
The overhang wasn't adequately supported, the perimeter had nothing to sit on, and the
nozzle started extruding into open air — the classic **"spaghetti"** failure, where the
print turns into a nest of loose filament.

**The fix: switch that part to tree supports.**

Tree supports branch in from the side and reach under overhangs, rather than growing
straight up as a solid block from the build plate. For an organic, curved geometry like
the elbow, they make contact where normal supports simply don't reach.

Worth flagging a subtlety I didn't appreciate at first: **the X2D's dedicated support
nozzle doesn't help here.** A second nozzle makes supports *easier to remove* and cleaner
at the interface — it doesn't change *whether* an overhang gets supported in the first
place. That's purely a function of support type and geometry. Dual extrusion is a
removal convenience, not a coverage guarantee.

If supports are new to you, [this video](https://youtu.be/89WspiTc5Z0) is a solid general
primer on when and why to use them.

<!-- TODO: note the exact STL filename of the elbow part, and add a photo of the
     spaghetti failure next to the successful tree-support print. Drop images in
     assets/images/ and reference them as ![alt](/assets/images/filename.jpg) -->

## The other lesson: buy more filament than you think

The guide estimates **$15–25** in filament, which reads like roughly one spool. It
wasn't enough for me.

I ran out mid-way through the part list with a single 1 kg spool, and since I hadn't
ordered a backup, the whole build stopped for **four days** waiting on delivery. Not a
hard problem — just an entirely avoidable one.

The arms alone are substantial, and once you add the base, the storage shell, the neck
and head, and the wheel base, one spool doesn't cover it. Matte filaments also tend to
be a little denser than standard PLA, so the same part weighs marginally more.

**Order two spools up front.** A spare spool of PLA costs about $20 and always gets
used eventually. Four idle days cost considerably more than that.

**And order the TPU95A at the same time.** The soft gripper fingers need it — PLA is
too rigid to work as a compliant gripper. It's easy to treat as an afterthought because
it's a small quantity and comes late in the part list, but forgetting it means a second
delivery wait on top of the first. Two materials, one order.

## Takeaways

- **Don't trust default supports on organic geometry.** Preview every part in the slicer
  before committing to a multi-hour run.
- **Tree supports for curved overhangs**, normal supports for flat ones.
- **Dual-nozzle support material solves removal, not coverage.** Different problem.
- **Dry the filament** if your air is humid — cheaper than a failed print.
- **Order two spools of PLA up front, plus the TPU95A.** One 1 kg spool did not cover
  this build, and the resupply cost me four days. The gripper needs TPU95A — order it
  in the same batch, not later.

## Next

Assembly, and finding out how much the tolerances actually matter.
