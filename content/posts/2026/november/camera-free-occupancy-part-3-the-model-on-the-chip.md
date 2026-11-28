---
title: "Part 3: Camera-Free Occupancy — The Model on the Chip, and a Screen That Glowed but Never Drew"
date: 2026-11-28T16:00:00+10:00
draft: false
slug: camera-free-occupancy-part-3-the-model-on-the-chip
cover:
  image: /img/camera-free-occupancy-part-3-present-on-pole.jpg
  alt: "A 2.8-inch display on a black pole showing PRESENT, 100% confidence, OCCUPIED and a small 8x8 heatmap"
  caption: "What you see walking in: the state the device decided on, how sure the model is, and what the time-of-flight sensor sees."
tags:
  [
    "Edge AI",
    "Embedded Rust",
    "Embassy",
    "STM32U585",
    "ST Edge AI",
    "CNN",
    "VL53L5CX",
    "AMG8833",
    "ILI9341",
    "SPI",
  ]
categories: ["Edge AI", "Tech"]
---

## Where Part 2 Left Off

In [Part 2](/posts/2026/november/camera-free-occupancy-part-2-the-dataset-is-done/) the dataset was frozen: 70 sessions over three days of a **VL53L5CX** time-of-flight sensor and an **AMG8833** thermal array looking down at my desk chair, two 8×8 grids per frame, three classes (`empty`, `seated`, `present`) that separate by eye. Part 2 promised the part all of that was for: train a model, score it honestly, and get it onto the STM32U585.

This post does that, then keeps going: the model running live on the board, the one posture it got wrong and how more data fixed it, and a small screen on the pole that shows what the device has decided. The screen took a whole Saturday of its own, and most of that day was spent on faults I'd seen before.

---

## Scoring It Honestly

The trap with this kind of data is the split. Frames 0.2 s apart are near-copies of each other, so if you split by frame, the test set is full of frames the model has effectively already seen, and the accuracy number is meaningless.

The unit I split on is a **round**: one daypart on one day, with every class recorded inside it. There are six. Train on five, test on the sixth, rotate, pool the six test sets. Every number below is on a day and daypart the model never saw.

| | accuracy | `present` recall | `present` called `seated` |
|---|---:|---:|---:|
| Logistic regression, 192 inputs | 98.2% | 94.7% | 500 frames |
| **Small CNN** | **99.1%** | **98.5%** | **80 frames** |

The CNN is deliberately tiny: two 3×3 convolutions, one pooling layer, two dense layers. **11,091 parameters.** No global pooling and no flip or shift augmentation, because *where* the warmth and the near returns sit is exactly what separates sitting from standing. The recipe was fixed before any held-out round was looked at, so the held-out data tunes nothing. Across three training seeds it scores **99.1% ± 0.01%**.

The number I care about more than accuracy: in the 40 sessions of an empty room, **0 of 24,772 frames were called occupied**. Almost all the remaining error sits at label boundaries, someone walking in at the edge of the thermal sensor's view, where even my own labels are a judgement call.

## Onto the Microcontroller

ST Edge AI Core turns the trained model into C: a network file, a 44 KiB weight array and a prebuilt runtime library. That goes into the Rust firmware the same way ST's sensor driver did in Part 1: compiled by `build.rs`, wrapped by `bindgen`, called through a small safe wrapper. It linked first time, which never happens.

| | |
|---|---|
| Flash | ~49 KiB of 2 MiB |
| RAM for activations | 5.75 KiB of 786 KiB |
| Work per inference | 186,787 multiply-accumulates |
| Time per inference | **8.2 ms** at 160 MHz |

The 8.2 ms took one fix. The first measurement was 18.5 ms, about 16 clock cycles per multiply-accumulate, far too slow. Embassy leaves the STM32U5's 8 KiB instruction cache **off**, so every instruction fetch waited on flash. Turning it on more than halved the time. At five frames a second the model uses about 4% of the CPU.

Then the check that matters most: does the board compute the same thing as the PC? The firmware logs the raw sensor grids alongside its own answer, and a script replays those grids through the PC model. Over a 10-minute live run, **3,255 of 3,255 frames gave the same class**, with the largest difference in the model's raw outputs at 1×10⁻⁵. Same encoding, same model, same answer.

## From Frames to a Decision

A model that answers five times a second will flicker at the edges: a frame of `present` in the middle of sitting, a frame of `empty` as you lean out of view. A building shouldn't see that. I chose the rule by replaying the model's held-out predictions through candidate rules rather than by guessing:

- **Nothing**, not even no debouncing at all, produced a single false empty↔occupied change in 2.9 hours of steady data. The occupied/vacant signal a building acts on was already clean.
- **Eight frames in a row** (1.6 s) removed almost all the seated↔present flicker, and every real change still showed within 2.6 s.

The live test then exposed a subtlety: walking in and sitting straight down, the class flickered between `seated` and `present`, and each flicker restarted the count. Occupied waited 4.2 s for the *class* to settle, even though both classes mean occupied. The fix was structural: **two debouncers**, one for occupied vs vacant and one for seated vs present. The worst entry dropped to 3.0 s.

## The Live Test, and the One Thing It Got Wrong

Ten minutes on the real board, scripted (walk in, sit, stand, lean, leave, walk past) and labelled afterwards from a webcam that stays on my machine:

| | |
|---|---|
| False empty↔occupied changes | **0** |
| Real ones caught | **12 of 12** |
| Walk-pasts shown as `seated` | **never** |

And one real error: about 16 seconds of `seated` while I stood **bent over the keyboard**. The model said `seated` on every single frame of it, so no debounce rule could hide it. From above, head and shoulders low over the seat look exactly like sitting, and the training data had no example of that posture labelled `present`.

That's a data problem, so the fix was data: one three-minute session standing bent over the desk. The old model called **all 938 frames** of it `seated`. Retrained with it, and scored on the live test, which no model ever trains on, the bent-over stretch went from **0% to 86–99% `present`** across three seeds. It wasn't free. Seated recall dropped by half a point, mostly in one session of hunched sitting. That was a trade I chose to take, and it's on the list to win back with more data.

---

## A Screen on the Pole

Gate 4's exit test is one person walking in and out of view and reading the result *off the device*. That needs a display: a 2.8″ ILI9341, 320×240, over SPI. The driver is hand-written, about a dozen controller commands with every one visible, drawing through `embedded-graphics`. The layout redraws only what changed: the state band on a change, the confidence on a change, and the 8×8 heatmap cell by cell. A typical frame costs well under a millisecond of SPI time. A state change costs about 0.2 s, once.

All of that was written and timed against the real SPI bus before the display arrived. Then I wired it.

### It Glowed, but Never Drew

![The display lying next to the board, backlight on, screen a uniform pale blue](/img/camera-free-occupancy-part-3-glowing-blank.jpg)
*Backlight on, nothing drawn. The board kept logging "drawn".*

The backlight came on, so it looked powered. Nothing appeared. Then the screen started cycling between dark, grey and bright in a slow rhythm. The board couldn't help: the display is write-only, so the firmware has no way to know whether anything arrived. It logs "drawn" either way.

So the board did what it could. A test program drove each signal line high on its own for five seconds while I followed with a multimeter, and the photo of the back of the module settled it:

![The display module's 14-pin header with its pin names printed above: the connector starts at RESET, one wire on GND, VCC and CS bare](/img/camera-free-occupancy-part-3-header-vcc-open.jpg)
*VCC (far left) and CS bare. The plug sat one position over.*

**VCC was bare, and so was chip select.** The module has two power pins that do different jobs. **LED** only switches the backlight. **VCC** feeds the regulator that powers the controller chip, the part that actually receives data and draws. With LED powered and VCC open, the panel lit, and the controller ran on whatever trickled in through its signal pins, which is why its shade followed those pins up and down.

The module prints every pin name right above its pin. "Wire label to label, never by counted position" is already the first rule in this project's notes, learned the hard way during the bring-up before Part 1, and I still lost an afternoon to it.

![The display showing HELLO in white with red, green and blue squares, a multimeter in the foreground](/img/camera-free-occupancy-part-3-hello.jpg)
*First pixels. Red, green and blue in the right order, so the colour setting was right first time.*

### The Sensor That Vanished

With the display working, the full program wouldn't start: **the thermal sensor had dropped off the I2C bus.** Its power LED was on. Continuity from its pins to the shield beeped. The time-of-flight sensor on the very same two wires answered fine.

The step that settled it: take the shield out of the question. With four jumper wires straight from the sensor to the board, it answered at once. The shield's 3.3 V reached it through a connection at a standoff that made contact some of the time: enough to light an LED, not enough for a chip to answer reliably. It's now a wire soldered straight to the 3.3 V rail, and I suspect it explains a few earlier "loose wire" moments too.

![The sensor head from below: the ST board with a proto shield carrying the time-of-flight and thermal sensor breakouts, and the display harness](/img/camera-free-occupancy-part-3-sensor-head.jpg)
*The sensor head. Time-of-flight top centre, thermal camera on the red board, display harness on the left.*

A third, smaller one for the collection: later the screen went white again with the wiring correct label for label. The Dupont leads had been turned 90° on their pins. A Dupont crimp is a little spring; twisted, it stops gripping while still looking seated. New leads fixed it.

### The Turn an Accelerometer Can't See

Taking the shield off and putting it back moved the sensor head. The model learned the room from one particular viewpoint, so this matters more than any wiring fault.

The board's own accelerometer measures the mount angle, and it said **56.6° tilt instead of 50.1°, with the roll 22° out**. I put both angles on the new display in large digits, green within a degree, so I could adjust the head at the pole instead of at the terminal. Easy.

Except an empty-room recording still didn't match. **36 to 49 of the 64 distance zones** had changed, when any two earlier empty-room recordings differ by 4 to 22.

An accelerometer measures gravity. Tilting forward or rolling sideways changes how gravity falls across its axes, but turning left or right about the vertical axis changes nothing, so it simply can't see it. The head was pointing about 10° to one side.

So I used the one reference that *can* see it: the room itself. The empty room, chair on its taped marks, looks the same in every reference session. A small alignment mode compares the live distance grid with that consensus and shows, on the display, how many zones disagree and how many columns the view is shifted. It started at about 40 zones and +2 columns. Turning the head brought it to 16 zones and a shift of 0, inside the normal spread between sessions.

The head settled at 45.4° / 167.2°, a few degrees from where the training data was recorded. Rather than chase the old pose and risk moving it again, I re-checked the model live in the new one: empty and seated steady at 96–99% confidence, and standing then bent over the keyboard read `present` for **17.6 s without a single break**. The display followed each change within 2 s. That pose is now the reference.

---

## On the Pole

![The display on the pole showing EMPTY, 100% confidence, VACANT](/img/camera-free-occupancy-part-3-empty-on-pole.jpg)
*Facing the door. The reader is the person arriving, not the person already in the chair.*

It faces the door rather than the chair, for two reasons. The question it answers, "is this desk free?", belongs to someone arriving. And a bright screen in the corner of a seated person's eye would change how they sit, which is the one thing the model depends on staying the same.

## What's Next

The Gate 4 exit test is what this display was built for: walk in, out and back, reading the result from the doorway. Then Gate 5 puts the decision where a building can use it: an MQTT message, a BACnet object (occupied / vacant, and confidence), and a panel in the same InfluxDB and Grafana stack my other sensors already feed.

The code, the dataset and the full lab notes, including every fault above in more detail, are on GitHub: [mapfumo/camera-free-occupancy](https://github.com/mapfumo/camera-free-occupancy).
