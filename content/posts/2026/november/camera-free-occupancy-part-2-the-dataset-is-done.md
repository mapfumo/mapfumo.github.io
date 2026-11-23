---
title: "Part 2: Camera-Free Occupancy — Doing It All Again, and the I2C Bus That Wouldn't Let Go"
date: 2026-11-24T07:45:00+10:00
draft: false
slug: camera-free-occupancy-part-2-the-dataset-is-done
cover:
  image: /img/camera-free-occupancy-part-2-grids.png
  alt: "Per-class average 8x8 grids from a time-of-flight sensor and a thermal array across 70 sessions: empty, seated and present"
  caption: "70 sessions, three days, every class in every daypart twice. Still separable by eye."
tags:
  [
    "Edge AI",
    "Embedded Rust",
    "Embassy",
    "STM32U585",
    "VL53L5CX",
    "AMG8833",
    "I2C",
    "time-of-flight",
    "thermal",
    "dataset",
  ]
categories: ["Edge AI", "Tech"]
---

## Where Part 1 Left Off

In [Part 1](/posts/2026/november/camera-free-occupancy-building-the-dataset/) I built the first half of a dataset for a room occupancy sensor that never sees a face: a **VL53L5CX** time-of-flight sensor and an **AMG8833** thermal array, two 8×8 grids, read by Rust/Embassy firmware on an STM32U585 and pointed down at my desk chair at 49.9°. Getting that far took the bring-up before it — ST's C driver linked over FFI, a header pinout that cost a full bench session, the mount angle measured by the board's own accelerometer — and by the end of Part 1 I had 34 sessions across morning, afternoon and evening, three classes (`empty`, `seated`, `present`) that separated by eye, and a decision about what an invalid ToF zone means to a model.

Part 1 ended with a caveat I couldn't wave away: **one day of each daypart**. If the afternoon's `seated` looks different from the morning's, is that the class, or was it just a warm Sunday? A single day can't tell a signal from that day's conditions. So the plan was simple and slightly tedious: do every round again, on a different day.

This post is that second pass — and the hardware fault it finally explained.

---

## Twelve Sessions, Three More Times

Each round is the same list, recorded the same way:

| Sessions | What's in view |
|----------|----------------|
| 5 × `empty` | Nobody. Chair on its floor marks, pushed in, pulled out 50 cm, turned 45°, removed |
| 3 × `seated` | Me, upright · leaning back · leaning in on my forearms |
| 3 × `present` | Standing at the keyboard · reaching across the desk · walking past (filmed, labelled frame by frame) |
| 1 × `empty` | Closing the round, in later light |

An afternoon round, an evening round, and a dawn round the next morning starting at 05:33 — within minutes of the first morning's start, so both mornings cover the same light. Thirty-six sessions, all recorded, checked, trimmed and committed one at a time. Total: **70 sessions** across three days, with every class in every daypart on two different days.

The rules from Part 1 still applied. Ten minutes empty before recording `empty` after anyone sat in the chair, because a seat stays warm. That rule caught me twice: once at the start of the morning round, and once after I'd sat down to label a walking session and needed the full 13½-minute settle before the last `empty`.

---

## What Repeated, and What Didn't

**The empty room is boringly consistent — good.** With the chair taken away entirely, one evening's empty scene differed from the previous evening's in **1 of 64** ToF zones. Chair pushed in: 2 of 64. The model gets the same background on different days, which is what it should learn to ignore.

**The turned chair wasn't.** "Turned 45°" on day two differed from day one in about **24 zones** — I'd turned it the other way, or further. My first instinct was to call that a mistake. It isn't: `empty` now covers two turned poses instead of one, and a chair's angle is exactly the kind of thing the model must learn is not a person.

**Posture signatures held.** Sitting upright puts near ToF returns along the bottom rows; leaning back moves them to the top — the pattern that broke the "seated = bottom-only" rule in Part 1 showed up again in every daypart. Reaching across the desk is all top rows, every time.

**Thermal contrast tracks the room, not the person.** A seated person's warmest zone, above the frame median:

| Daypart | Day 1 | Day 2 |
|---------|------:|------:|
| Morning | +5.0 to +5.5 °C | +4.5 to +4.9 °C |
| Afternoon | +3.0 to +3.25 °C | +2.25 to +2.75 °C |
| Evening | +3.25 to +3.5 °C | +3.5 °C |

Same body, half the contrast on a warm afternoon. On those afternoons the ToF carries more of the load — which is the whole argument for fusing two sensors rather than trusting either.

---

## Evening Light, Done Honestly

Before the second evening round I asked a simple question: what time did we start yesterday? The dataset deliberately stores no wall-clock times (Part 1, privacy), but git does — every session was committed a minute after it finished.

The commit log told an uncomfortable story. Day one's evening round started at about **18:17**, four minutes before sunset. The `empty` sessions ran through sunset and twilight; every `seated` and `present` session came after dark. Light level lined up with class.

It's a small effect — daylight changed only about 5 of 64 ToF zones in the morning test from Part 1 — but it's exactly the kind of shortcut a model will happily learn: "dim room means someone's there". I kept the same start time for consistency, then made sure the round ended on an `empty` session around 20:10, in full dark. Across the two evenings, `empty` now covers twilight *and* darkness.

The lesson I'm keeping: a confound doesn't need to be large to matter. It only needs to line up with the label.

---

## Two Ways to Walk Past

Walking past is the one class where the session label lies — a person is in view only part of the time — so those sessions are filmed and labelled frame by frame, with the video never leaving my machine (Part 1 has the method). Three more were added, one per daypart, and two of them came out very differently.

On the afternoon I made **quick, straight passes** to and from another room. A third of the frames are `present`, and nearly all of those carry an unmistakable signature.

On the evening I **deliberately walked away slowly** from the desk. Half the frames are `present` — but a quarter of those sit just under my crude thresholds: nearest ToF return **0.89–0.97 m**, where the empty room starts at about 1 m, and thermal **+1.0 to +1.5 °C**, just under the +1.5 °C cutoff.

I checked those frames before trusting the labels. None of the `empty` frames in that session showed any sign of a person at all, and the weak `present` frames came in short runs every ten seconds or so — the start and end of each pass. They're real. They're the faint edge of a person, and they're the most useful frames in the session: a dataset of only crisp passes would teach the model that `present` always means a strong signal.

---

## The I2C Bus That Wouldn't Let Go

Since bring-up, a session would occasionally refuse to start. The firmware probed the I2C bus for a sensor — usually the thermal one — found nothing, and gave up. A rescan a minute later always found it. Part 1 blamed a loose jumper wire.

Then I remembered there aren't any. Both sensors sit on a proto shield plugged straight into the board's Arduino header, with the wiring soldered in place. The third dropout, on the second afternoon, cost a full 13½-minute recording with zero frames, and that was enough to look properly.

![The sensor board on its boom seen from below, with the thermal and time-of-flight breakouts on the shield and the USB cable running down the pole](/img/camera-free-occupancy-rig-from-below.jpg)
*Both sensors share one I2C bus on the shield. The braided USB cable running down the pole is the one I knocked loose in Part 1.*

The better suspect was how every session *ends*. I stop a recording by killing the host process on a timer, which halts the microcontroller wherever it is — sometimes halfway through reading a byte from a sensor. I2C is a shared two-wire bus. A sensor that is in the middle of sending a 0 bit pulls the data line (SDA) low and keeps it there, patiently waiting for clock pulses that will never come. The next session starts, sees a busy bus, and concludes nobody's home.

The textbook fix is a **bus clear**: before the I2C peripheral claims the pins, drive the clock line by hand until the sensor finishes its byte and lets go, then send a STOP. A sensor that was idle ignores clocks with no START, so it's safe to do on every boot. I added it to the firmware with three probe attempts, and made it log only when it actually does something — so the next failure would say whether the theory was right.

The next morning, session 3 started with this:

```
I2C1 bus recovery (attempt 1): sda_was_stuck: true, pulses: 1, bus_free: false
no AMG8833 on attempt 1 of 3
I2C1 bus recovery (attempt 2): sda_was_stuck: true, pulses: 6, bus_free: true
AMG8833 found on attempt 2 of 3
```

The theory, confirmed in the act — and a bug in my fix, found the same way. The first attempt stopped clocking as soon as SDA read high, after one pulse. But a sensor mid-byte will happily put a 1 bit on the line, then pull it low again for the next bit. "SDA went high" doesn't mean "the sensor let go". The retry is what saved the session.

The corrected version always sends all **nine** clock pulses when SDA is stuck — enough to finish any byte plus its acknowledge slot — then the STOP. I haven't seen it clear a real stuck bus yet; ten deliberately killed restarts afterwards all came up clean. It still logs when it acts, so the next one will tell me.

Neither this nor the USB cable I knocked out mid-session in Part 1 matters at a desk with someone there to press go again. Both would matter in the multi-day unattended run planned later in the project, which is why the recovery lives in the firmware rather than in my head.

---

## The Dataset Is Done

![Per-class average grids across all 70 sessions](/img/camera-free-occupancy-part-2-grids.png)

The same picture as Part 1, now from 70 sessions. `empty` is flat in thermal with the ToF seeing only the room beyond 1 m; `seated` is a warm block over near returns along the bottom rows; `present` moves the warmth upper-left and fills the top of the ToF grid.

Share of frames with a person signature, by daypart:

| Class | Morning | Afternoon | Evening | Frames |
|-------|--------:|----------:|--------:|-------:|
| `empty` | 0% | 0% | 1.9% | 27,381 |
| `seated` | 100% | 100% | 100% | 16,777 |
| `present` | 98.9% | 99.2% | 97.9% | 13,358 |

The `empty` 1.9% is still the warm-seat session from Part 1; the `present` shortfall is the faint pass edges above. **57,516 labelled frames.** And the zero that lied in Part 1 didn't go away with more data: 1,364 "valid" 0 mm readings now, 1,294 of them in that same corner zone, 1,192 in `empty` frames. The encoding that masks them out was the right call.

That meets the bar I set at the start — the classes are separable by eye — so the dataset is tagged and frozen. The caveats travel with it:

- **One person, one room, one chair.** Whatever the model learns, it learns about me at this desk.
- **Frames within a session are near-duplicates.** This is 70 situations, not 57,000 independent observations. Train and test must be split by *session*; split by frame and the test set is full of near-copies of training frames, and the accuracy number means nothing.
- **The classes are imbalanced**, roughly 2 : 1.2 : 1.
- **One open question:** is "occupied a minute ago" — a warm empty seat — an error, or a feature?

---

## What's Next

Part 3 is the part all of this was for: training a model in STM32Cube AI Studio, scoring it honestly on sessions it has never seen, and getting it onto the microcontroller.

The code, the dataset and the full lab notes are on GitHub: [mapfumo/camera-free-occupancy](https://github.com/mapfumo/camera-free-occupancy).
