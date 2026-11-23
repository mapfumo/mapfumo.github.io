---
title: "Part 1: Teaching a Camera-Free Sensor to Tell Empty from Seated — 34 Sessions, Three Dayparts, and a Zero That Lied"
date: 2026-11-23T07:30:00+10:00
draft: false
slug: camera-free-occupancy-building-the-dataset
cover:
  image: /img/camera-free-occupancy-class-grids.png
  alt: "Per-class average 8x8 grids from a time-of-flight sensor and a thermal array: empty, seated and present"
  caption: "Two 8×8 sensors, no camera. Three classes, separable by eye across morning, afternoon and evening."
tags:
  [
    "Edge AI",
    "Embedded Rust",
    "Embassy",
    "STM32U585",
    "VL53L5CX",
    "AMG8833",
    "time-of-flight",
    "thermal",
    "dataset",
    "privacy",
  ]
categories: ["Edge AI", "Tech"]
---

## The Setup

I'm building a room occupancy sensor that never sees a face. No camera, no images — just two 8×8 sensors pointed at a desk and a chair:

- a **VL53L5CX** time-of-flight sensor, which measures distance in 64 zones;
- an **AMG8833 Grid-EYE**, which measures radiated heat in 64 pixels.

Both sit on one I2C bus on a **B-U585I-IOT02A** (STM32U585, Cortex-M33), running Rust firmware on Embassy. ST's C driver for the ToF is linked in over FFI. The end goal is a small model running on the microcontroller that says *empty*, *seated* or *present*, feeding the same BACnet/MQTT/Grafana pipeline as my earlier projects.

Before any of that, the model needs data. This post is about collecting it — 34 sessions over two days — and the handful of things the sensors told me that I would not have guessed.

The rule I set myself: **don't train anything until the classes are distinguishable by eye as heatmaps.** If a person can't see the difference between empty and seated in the raw grids, no model is going to fix that. The mounting is.

---

## Three Classes, Decided by Geometry

The rig is clamped to the back of the desk, pointing down at the chair. I measured the angle with the board's own accelerometer — **49.9° below horizontal**. A tape measure failed: furniture hides the floor, and a warm seat looks like the target.

That angle decided the class set. At 49.9° down, a standing person's head is out of frame, so "standing" and "walking past" look the same from the sensor's point of view. Three classes, not four:

| Class | Meaning |
|-------|---------|
| `empty` | Nobody in view |
| `seated` | One person in the chair |
| `present` | Someone in view but not seated — standing at the desk, reaching across, walking past |

The honest product claim that falls out of that: **workstation occupancy**, not room counting.

---

## The Collection Rules Came from Measurements

Every rule in the collection procedure exists because a measurement forced it.

**Wait 10 minutes before recording `empty`.** A seat stays warm. After a 10.6-minute sit, the warmest seat zone was +1.25 °C over baseline the moment I stood up and took about 7 minutes to fall back to the noise floor. Record `empty` straight after `seated` and you teach the model that a warm seat means nobody's there — or worse, that it means somebody is.

**Vary the chair on purpose.** The chair is part of the scene. I taped marks on the floor and found the chair still returns ~10 cm differently each time — a one-column shift in the ToF grid. Rather than pretend it's fixed, every `empty` session records the chair in a different state: on its marks, pushed in, turned 45°, pulled out, gone. The model has to learn that a chair's position is not a person.

And one surprise: **a black leather chair pulled 50 cm back is invisible to the ToF.** Dark leather at ~1.5 m returns nothing at all. The same thing turned out to be the most useful signal in the dataset — more on that below.

**Watch for the rig moving.** The accelerometer logs tilt on every frame. Footsteps make the boom wobble — single frames swung from 48.05° to 51.27° while someone walked past — so the drift check compares 5-second medians, which stayed within 0.1°. A real knock of 2.2° gets caught in 2.4 seconds.

---

## A Camera — But Only to Label

Here's the awkward bit for a camera-free project: some frames need a camera to label them.

A "walking past" session is only partly `present`. In one session a person was in view in just 54% of frames — 21 passes, with empty room in between. Label the whole session `present` and the model learns that an empty room is sometimes present.

So the webcam on my monitor records alongside the walking sessions, synced to the sensor stream to within about one frame (offset −0.33 s, checked by eye at sharp onsets). I step through every frame in a small labelling tool with the heatmaps beside the video.

Two things made this less straightforward than it sounds:

1. **Video motion doesn't predict sensor presence.** The webcam sees the whole room; the sensor sees only the desk and chair from above. Correlation between whole-frame webcam motion and the sensor signature: roughly zero. A person the camera sees clearly can still be an `empty` frame. The question is always *"is someone in the sensor's view?"*
2. **The tool's pre-fill can't be trusted.** It suggests labels from the sensor signature, but it never suggests `seated`, and a seat warmed by someone who just stood up reads as `present`. Every frame gets checked by eye — otherwise the model just learns the pre-fill's rule.

The footage never leaves my machine. It's git-ignored; only the derived labels are committed. The dataset itself stores no wall-clock times either — just seconds since a session started and a coarse daypart (`am`, `pm`, `eve`).

---

## Three Dayparts, One Picture

I recorded the same protocol three times: afternoon, evening, and this morning from 05:30. Every class, several chair states, 2.5–3.5 minutes per session at 5 Hz.

![Per-class average grids across all three dayparts](/img/camera-free-occupancy-class-grids.png)

Read it row by row:

- **`empty`** — thermal is flat (within ±0.6 °C); the ToF only sees the far room at 1.0–1.7 m.
- **`seated`** — a warm block lower-left, and near returns at 0.6–0.8 m along the bottom rows. Most of that ToF signal is the empty black chair back returning *nothing*, and a body in front of it returning ~0.5 m.
- **`present`** — the warmth moves upper-left and the near returns fill the top. Standing puts the body higher in the view and nearer the board.

The same story as one number per frame — share of frames with a person signature:

| Class | Morning | Afternoon | Evening |
|-------|--------:|----------:|--------:|
| `empty` | 0% | 0% | 4.3% |
| `seated` | 100% | 100% | 100% |
| `present` | 100% | 100% | 100% |

The 4.3% is real, and instructive: 165 frames from one evening session, labelled empty by eye, where the seat is still warm from someone who just got up. Thermal only, no near ToF returns. Whether "occupied a minute ago" should count as a feature is a decision I now have data for.

The dayparts differ in ways that matter. The morning room was **4 °C cooler** than the evening, so the same body stood out more — a seated person read +5 °C over the frame instead of +3 °C. Evening lighting lets the ToF reach the far wall at 1.6–2.0 m, where in afternoon daylight those zones returned nothing. A model trained on one daypart alone would have learned those zones as always-invalid.

What *didn't* matter: daylight. I ended the morning round with one more empty session, recorded in full daylight 1 h 40 min after the dawn one. Only **5 of 64** ToF zones differed. Ending the round on an `empty` was cheap insurance — otherwise `empty` would only exist in the dimmest light, and brightness would quietly track the class.

---

## The Zero That Lied

Before training, I had one decision left: what does an invalid ToF zone mean as model input?

It sounds like housekeeping. It isn't. Across all 27,383 labelled frames, roughly a third of zones are "invalid" — and "invalid" turns out to be several different things:

| ToF status | Share of zones | What the distance actually is |
|-----------|---------------:|-------------------------------|
| 5 — valid | 62.7% | Real. Frame-to-frame jitter ~9 mm |
| 255 — no target | 29.3% | **Not a measurement** — 85% of the time it repeats the previous frame's value |
| 10 — valid, none last frame | 5.8% | Roughly right but noisy; mostly the far wall flickering |
| 4 — consistency failed | 1.2% | Unreliable — 10% of readings off by >630 mm |

Then the one that matters. **1,070 readings came back with a "valid" status and a distance of 0 mm.** Zero is physically meaningless — nothing is touching the cover glass. And they weren't random: 1,000 of them sit in a single corner zone, and 1,018 are in `empty` sessions.

Fed in as-is, that's a gift-wrapped shortcut: *"something touching the sensor in the bottom-right corner means the room is empty."* The model would score well and have learned nothing about people.

The encoding I settled on gives each frame three 8×8 channels:

1. **ToF valid mask** — 1 only if the status is 5 or 9 *and* the distance is above zero.
2. **ToF distance** — clipped to 2 m and scaled to 0–1, with invalid zones filled as **far**. Smaller always means nearer; "nothing came back" never reads as "something at the glass".
3. **Thermal** — each pixel's °C above the frame median.

Why not just drop invalid zones, or fill them with zero? Because in this pose "no return" *is* the empty-chair signature. Erase it and you erase most of the ToF's ability to see someone sit down. The mask lets the model tell "I measured the far wall" from "nothing came back at all" — 64 extra inputs, which is nothing for a model this size.

---

## What Went Wrong

Not everything ran clean this morning:

- **A sensor went missing at startup.** One session stopped immediately with no AMG8833 on the bus; a rescan a minute later found it. Yesterday the ToF did the same thing once. One dropout per sensor, both gone on retry — that points at a jumper wire, not a breakout.
- **The debug probe dropped off USB** 144 seconds into a session while I was reaching across the desk. The kernel logged a protocol error and re-enumerated a second later. I kept the 2.2 minutes recorded before it.

Neither matters at a desk with someone there to retry. Both would matter in the multi-day unattended run planned for later, so both are on the list.

---

## What's Next

The data says the three classes separate — across both sensors, three dayparts and five chair states. But the caveats are real:

- **One person, one room, two days.** Frames within a session are near-duplicates, so this is 34 situations, not 27,000 independent observations.
- I still want a **second day** across all dayparts, and an afternoon walking-past session with video.

Then the dataset gets tagged, and the next post is the part this was all for: training a model in STM32Cube AI Studio and running it on the microcontroller.

The code, the dataset and the full lab notes are on GitHub: [mapfumo/camera-free-occupancy](https://github.com/mapfumo/camera-free-occupancy).

**Next:** [Part 2](/posts/2026/november/camera-free-occupancy-part-2-the-dataset-is-done/) — the second day of every daypart, the finished 70-session dataset, and the I2C bus that wouldn't let go.
