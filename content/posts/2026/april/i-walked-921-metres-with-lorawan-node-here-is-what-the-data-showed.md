---
title: "921 Metres Through Suburban Brisbane with a LoRaWAN Node — Here's What the Data Showed."
date: 2026-04-28T12:23:09+10:00
draft: false
cover:
  image: /img/lorawan-walk-test.svg
  alt: "LoRaWAN walk test — elevation profile with colour-coded signal quality across 921 metres"
  caption: "246 uplinks logged. Elevation beat distance every time."
tags:
  [
    "LoRaWAN",
    "field-test",
    "AU915",
    "RSSI",
    "SNR",
    "STM32WL55",
    "Grafana",
    "influxdb",
    "range-testing",
  ]
categories: ["IIoT", "Tech"]
---

## The Setup

I built a two-node LoRaWAN sensor network on STM32WL55JC1 microcontrollers running Rust/Embassy firmware — the full story is in [Part 1](/posts/2026/april/building-a-lorawan-sensor-network-in-rust-stm32wl55-embassy-and-the-bugs-that-taught-the-most/). One of the two nodes, lora-2, carries no sensor. Its only job is to send a confirmed uplink every ~10 seconds, display RSSI, SNR, and DR on its OLED, and let the gateway's InfluxDB record tell the story later.

You might wonder why I didn't just use lora-1 — the sensor node — for this test. Both nodes send a 4-byte payload, so packet size is identical and makes no difference to range. What matters is transmission behaviour: lora-2 sends every uplink confirmed, so a missed ACK is detected within one cycle. lora-1 only confirms every fifth uplink, meaning gateway loss takes much longer to register. lora-2 also transmits every ~10 seconds versus lora-1's ~30 seconds — three times the data points in InfluxDB, three times the resolution when overlaying against GPS waypoints later. Purpose-built beats repurposed.

With the network running reliably, it was time to take lora-2 outside and find out what "range" actually means in a real suburban environment.

The gateway — a RAK7268V2 WisGate Edge Lite 2 — went on the roof. Altitude 85 metres. AU915, sub-band 1, SF12.

I grabbed my phone running GPS Logger, noted the start time, and walked.

---

## The Method: Record Nothing, Measure Everything

This is the part that surprised people when I described it.

I did not manually record RSSI, SNR, or DR at each waypoint. I did not carry a notebook. I did not try to read and write down numbers while walking.

InfluxDB was doing that for me. Every confirmed uplink from lora-2 hit the gateway, got decoded by the MQTT bridge, and landed in InfluxDB with a precise timestamp. 246 records over the 70-minute walk.

All I needed in the field was **GPS coordinates and time** at each waypoint. A screenshot of GPS Logger took three seconds. Back at the desk, I overlaid the timestamps against the InfluxDB data in Grafana and got the full picture.

The field notes look like this:

```text
09:40  Home / gateway (roof)
10:08  WP2 — short walk out
10:15  Local church (~256 m)
10:22  WP4 (~380 m)
10:26  WP5 (~466 m)
10:31  WP6 (~593 m)
10:37  WP7 — downhill (~716 m)
10:42  WP8 — back uphill (~743 m)
10:50  WP9 — furthest point (~921 m)
```

That is it. No signal values in the field at all.

---

## The Results

All signal values below are **gateway SNR** — what the RAK7268V2 received from lora-2. This is the uplink path: node → gateway. The OLED on the board shows downlink SNR (gateway → node), which is a different measurement on a different frequency with different bandwidth. [NOTES.md](https://github.com/mapfumo/lorawan/blob/main/NOTES.md) explains the difference if you want the detail.

| Waypoint       | Time  | Distance | Altitude | RSSI     | SNR      | Status              |
| -------------- | ----- | -------- | -------- | -------- | -------- | ------------------- |
| Home / Gateway | 09:40 | 0 m      | 63 m     | −17 dBm  | +13.8 dB | Excellent           |
| WP2            | 10:08 | 63 m     | 77 m     | −70 dBm  | +9.5 dB  | Good                |
| Local church   | 10:15 | 256 m    | 89 m     | −89 dBm  | +4.2 dB  | Good                |
| WP4            | 10:22 | 380 m    | 87 m     | −102 dBm | −6.5 dB  | Marginal            |
| WP5            | 10:26 | 466 m    | 82 m     | −103 dBm | −19.5 dB | **At limit**        |
| WP6            | 10:31 | 593 m    | 75 m     | −103 dBm | −20.8 dB | **Packet loss**     |
| WP7 — downhill | 10:37 | 716 m    | 48 m     | −103 dBm | −19.2 dB | **Terrain masking** |
| WP8 — uphill   | 10:42 | 743 m    | 64 m     | −96 dBm  | +0.5 dB  | **Recovered**       |
| WP9            | 10:50 | 921 m    | 77 m     | −103 dBm | −13.2 dB | Marginal            |

SF12 (DR0) can theoretically decode down to **−20 dB SNR**. WP5 hit −19.5 dB and WP6 hit −20.8 dB — right at the wall. The large packet gaps in the InfluxDB data between WP6 and WP8 (354 seconds of silence, then 242 more) confirmed it. The node was transmitting; the gateway could not decode it.

---

## The Finding That Changed How I Think About Gateway Placement

Look at WP7 and WP8 side by side.

- **WP7**: 716 metres from the gateway, altitude 48 metres → SNR −19.2 dB, 354-second gap
- **WP8**: 743 metres from the gateway, altitude 64 metres → SNR +0.5 dB, signal recovered

WP8 is _further away_ and has _better signal_. The only difference is altitude — 16 metres higher.

I had walked downhill into a dip. At WP7, I was 37 metres below the gateway rooftop. The terrain between us clipped the Fresnel zone — the elliptical volume of space the radio wave occupies between transmitter and receiver. At 700 metres on 915 MHz, the first Fresnel zone radius at the midpoint is roughly 14 metres. Terrain does not need to block the geometric line-of-sight to degrade the link; it only needs to intrude into that zone.

Climbing 16 metres at WP8 restored line-of-sight to the gateway roof. SNR jumped from −19 dB to +0.5 dB. Distance was irrelevant. **Elevation was everything.**

---

## What the Packet Gaps Tell You

The InfluxDB frame count is the most useful signal quality indicator during a range test — more useful than RSSI or SNR alone, because it shows actual packet loss rather than received signal strength.

Significant gaps during this walk:

| Time (AEST)   | Gap   | Last SNR before gap | Interpretation                       |
| ------------- | ----- | ------------------- | ------------------------------------ |
| 09:50 → 09:55 | 307 s | +14.2 dB            | Gateway moving to roof — expected    |
| 10:30 → 10:36 | 354 s | −20.8 dB            | Below SF12 decode limit — heavy loss |
| 10:36 → 10:40 | 242 s | −19.2 dB            | Downhill, Fresnel zone blocked       |
| 10:50 → 10:54 | 254 s | −3.0 dB             | Furthest point, marginal path        |

The 307-second gap at 09:50 is not a coverage failure — it was the gateway rebooting after being moved to the roof. The signal before and after that gap was excellent (SNR +14 dB). The data captures the whole story, including things I would not have noticed in the field.

---

## What This Means in the Field

This test was run in a hilly Brisbane suburb — houses, fences, trees, and undulating terrain at every step. It is a realistic worst case for LoRaWAN.

The numbers:

- **Reliable range** (SNR consistently above 0 dB): ~350–400 metres in this terrain
- **Marginal range** (packets received but gaps present): 400–950 metres
- **Maximum observed range**: 921 metres, still receiving packets

For context, AU915 SF12 in open flat rural terrain with a gateway on a silo or water tower — elevated 20–30 metres above ground — will typically reach 5–15 kilometres. The suburban environment with its buildings and terrain introduces 10–15 dB of additional path loss that simply does not exist in open paddocks.

**The key insight for anyone deploying LoRaWAN in agriculture:** gateway placement on the highest available structure is the single biggest range multiplier. Not a better antenna. Not a higher spreading factor. The elevation of the gateway itself. A gateway mounted 10 metres higher covers a dramatically larger area, especially across hilly terrain where Fresnel zone clipping is the dominant loss mechanism.

---

## The Methodology Works

The GPS-plus-InfluxDB approach proved itself. I spent the walk focused on moving to the right locations and noting the time — not squinting at numbers on the OLED and writing them down. The data in InfluxDB was richer than anything I could have captured manually: every uplink timestamped to the second, every gap documented, the full arc from 09:35 to 10:57 captured without gaps.

If you are doing LoRaWAN range testing, this is the method:

1. Start the backend before you leave
2. Confirm data is flowing in Grafana
3. Note GPS coordinates and time at each waypoint — a phone screenshot takes three seconds
4. Walk
5. Analyse in Grafana when you get back

The field is for walking. The data analysis is for the desk.

---

## What's Next

The next test will be in open rural terrain — the environment the agriculture use case actually requires. I expect to see 5–10× the range demonstrated here, with the elevation effect even more pronounced across flat paddocks where the gateway's height advantage translates directly into kilometres of additional coverage.

The full source — firmware, backend, MQTT bridge — is on [GitHub](https://github.com/mapfumo/lorawan). The detailed signal analysis with gap tables and the Fresnel zone explanation is in [NOTES.md](https://github.com/mapfumo/lorawan/blob/main/NOTES.md).
