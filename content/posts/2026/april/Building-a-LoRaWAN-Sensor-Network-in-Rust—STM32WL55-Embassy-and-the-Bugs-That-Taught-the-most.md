---
title: "Building a LoRaWAN Sensor Network in Rust — STM32WL55, Embassy, and the Bugs That Taught Me the Most"
date: 2026-04-28T12:21:14+10:00
draft: false
cover:
  image: /img/lora-boards.jpg
  alt: "Two STM32WL55JC1 NUCLEO boards in enclosures — lora-1 (sensor node) and lora-2 (range probe)"
  caption: "lora-1 and lora-2 — both running Rust/Embassy firmware over AU915 LoRaWAN"
tags: ["Rust", "Embedded Rust", "LoRaWAN", "Embassy", "STM32", "STM32WL55", "no_std", "influxdb", "Grafana", "MQTT"]
categories: ["IIoT", "Tech"]
---

## From Sensor to Dashboard — Without a Line of C

My previous projects used the STM32F446RE — a capable workhorse, but one that needs an external LoRa radio module wired up separately. When I decided to go deep on LoRaWAN, I wanted a chip where the radio was part of the silicon itself. The STM32WL55JC1 delivers exactly that: an ARM Cortex-M4 application processor and a sub-GHz radio coexisting on the same die, talking to each other over an internal SPI bus that never leaves the package.

The goal was a complete, end-to-end LoRaWAN sensor network:

- Two nodes sending real data over the AU915 band to a RAK7268V2 gateway
- A Docker backend (InfluxDB + Grafana + MQTT bridge) ingesting and visualising every uplink
- Everything written in Rust — `no_std`, no heap, no OS

What followed was one of the most instructive builds I've done. Not because it went smoothly. Because it didn't.

---

## The Hardware

Both boards are NUCLEO-WL55JC1 development boards — STM32WL55JC1 plus an SH1106 128×64 OLED display wired to the I2C2 bus.

| Board | Role | Sensor | Payload |
| --- | --- | --- | --- |
| lora-1 | Environmental sensor | SHT41 (temperature + humidity) | 4 bytes |
| lora-2 | Range probe | None | TX counter only |

lora-2 carries no sensor by design. Its job is to send confirmed uplinks as fast as the duty cycle allows and report RSSI, SNR, and DR on the OLED. When I take it outside, it becomes a live signal quality instrument. More on that in a [follow-up post about the field test](/posts/2026/april/i-walked-921-metres-with-lorawan-node-here-is-what-the-data-showed/).

---

## The Firmware Stack

```text
Language:    Rust (no_std, no_main)
Runtime:     Embassy 0.7 (async/await on bare metal)
LoRa PHY:    lora-phy 3.0 (patched — see below)
LoRaWAN MAC: lorawan-device 0.12 (patched)
Target:      thumbv7em-none-eabihf (Cortex-M4, no FPU)
Flash tool:  probe-rs via .cargo/config.toml
```

### Why Embassy?

Embassy replaces a traditional RTOS with Rust's native `async/await`. Instead of task scheduling, preemption, and mutexes, you write cooperative coroutines that suspend at `await` points and resume when the hardware is ready. For this workload — wait for LoRa TX, wait for sensor measurement, wait for a timer — it maps naturally:

```rust
// Suspend this task until the radio finishes — no busy-wait, no blocking
device.send(&payload, 1, true).await?;

// Wait for the SHT41 measurement cycle
Timer::after_millis(10).await;
```

No heap required. Async state machines are stack-allocated. No context switch overhead. If it compiles, a large class of memory safety bugs is already gone.

---

## Constraint 1: No FPU

The STM32WL55 has no floating-point unit. Despite the `eabihf` (hard-float ABI) in the target triple — which is required by the LoRa PHY crate — the chip cannot execute FPU instructions at runtime.

This means every temperature and humidity calculation has to use integer math with fixed-point scaling. The SHT41 gives raw ADC counts; the datasheet formula normally produces a float. Instead:

```rust
// Temperature: multiply by 175, divide by 65535, subtract 45 — all integer
let temp_raw: u16 = read_sht41();
let temp_x100: i32 = (temp_raw as i32 * 17500 / 65535) - 4500;

// temp_x100 = 2350 means 23.50 °C
// Encode into payload as i16 (×100 implicit)
payload[0] = (temp_x100 >> 8) as u8;
payload[1] = (temp_x100 & 0xFF) as u8;
```

The MQTT bridge on the server side divides by 100 to recover the float for InfluxDB. No FPU required anywhere in the chain.

**Lesson:** Fixed-point integer math is not a limitation — it is a discipline. It forces you to think explicitly about precision and range at every step, which is exactly the kind of thinking embedded systems demand.

---

## Constraint 2: The Patched Crates Problem

`lora-phy` and `lorawan-device` are the two crates that handle the LoRa radio and LoRaWAN MAC layer. Both are published on crates.io — but both depend on an older `embassy-time` API that broke when Embassy moved to version 0.4.

The symptom: `cargo build` fails with type errors buried in the dependency tree. The crates compile fine on their own; they break when combined with a current Embassy workspace.

The fix: fork both crates locally, update the `embassy-time` API calls, and point `Cargo.toml` at the local patches:

```toml
[patch.crates-io]
lora-phy = { path = "../lora-phy-patched" }
lorawan-device = { path = "../lorawan-device-patched" }
```

The patches are minimal — a handful of renamed methods in `embassy-time` — but finding them required reading the Embassy changelog, the crate source, and several GitHub issues across three repositories.

**Lesson:** The embedded Rust ecosystem is maturing fast, but crate versions lag behind Embassy releases. Pin your dependencies and do not `cargo update` without a full test cycle on hardware.

---

## Bug 1: The I2C Bus Contention Nightmare

The SH1106 OLED and the SHT41 sensor share the I2C2 bus. On lora-1, this worked perfectly from day one. On the original lora-2 (which used a BME688 instead of an SHT41), the display was corrupted every single time.

The symptom: the first two lines of the OLED displayed correctly. Everything below was garbage. The sensor read fine. The display looked like a scrambled TV channel.

Attempted fixes, in order:

1. `bus_recover()` — toggle SCL nine times to reset a stuck device. No effect.
2. Initialise sensor after display. Same corruption.
3. Use one I2C instance per call instead of per transaction. Worse.
4. Write a custom raw I2C driver with 8-byte maximum writes. Worked in isolation, failed in the main loop.

The root cause took three days to find. The BME688 has a hardware quirk: during its internal boot sequence on power-up, it holds SDA low mid-transaction. The SH1106's `flush()` method sends the entire 1 KB framebuffer in a single I2C transaction. When the BME688 asserted SDA during that burst, the OLED received a NACK after ~32 bytes and stopped writing — which is exactly where the corruption began.

**The fix:** replace the BME688 with an SHT41. The SHT41 never holds SDA unexpectedly. The bus stayed clean and `flush()` worked every time.

**Lesson:** When two I2C devices share a bus and one is pathologically badly behaved at power-up, the only reliable fixes are hardware isolation (separate bus) or removing the offending device. Software bus recovery cannot fix a device that re-asserts the bus between recovery pulses.

---

## Bug 2: Gateway Loss Detection — The One That Really Stung

This bug looked like a solved problem. It wasn't.

When the gateway is powered off, LoRaWAN nodes have no passive notification — the radio is fire-and-forget on uplinks. I added a counter: if three consecutive uplinks fail, mark the node as disconnected and trigger a rejoin.

It didn't work. Power off the gateway, and lora-2 sat on `Connected` indefinitely.

The problem was in the match arm:

```rust
// First attempt — BROKEN
match device.send(&payload, 1, true).await {
    Ok(response) => {
        uplink_fail_count = 0;  // resets even on NoAck!
        // ...
    }
    Err(e) => {
        uplink_fail_count += 1;
    }
}
```

I assumed that a missed ACK would come back as `Err`. It does not.

The `lorawan-device` crate returns `Ok(SendResponse::NoAck)` when a confirmed uplink times out with no ACK in RX1 or RX2. `Err` is reserved for radio hardware faults — transceiver failures, SPI errors, things that are genuinely broken at the silicon level. A missing gateway is not an error to the radio; the uplink was transmitted successfully. The network simply didn't respond.

```rust
pub enum SendResponse {
    DownlinkReceived(mac::FcntDown),
    SessionExpired,
    NoAck,      // confirmed uplink sent, gateway did not ACK — not an Err
    RxComplete, // unconfirmed uplink complete — no gateway feedback at all
}
```

The fix is explicit matching on `NoAck`:

```rust
match device.send(&payload, 1, use_confirmed).await {
    Ok(SendResponse::NoAck) => {
        uplink_fail_count += 1;
        if uplink_fail_count >= MAX_UPLINK_FAILS {
            is_joined = false;  // trigger rejoin
        }
    }
    Ok(_) => {
        if use_confirmed { uplink_fail_count = 0; }
    }
    Err(e) => {
        // radio hardware fault — handle separately
    }
}
```

Power off the gateway now and lora-2 shows `Connecting...` within three uplink cycles (~30 seconds). Power it back on and the board rejoins automatically within the backoff window.

**Lesson:** In LoRaWAN, "uplink sent without error" and "gateway received the uplink" are not the same thing. Read the enum variants. Do not assume that a missing ACK maps to `Err`.

---

## The Backend Pipeline

The backend runs entirely in Docker Compose:

```text
RAK7268V2 gateway
    └── MQTT broker (built-in, port 1883)
           └── mqtt_to_influx.py (Python bridge)
                  └── InfluxDB 2.x
                         └── Grafana (10s refresh)
```

The MQTT bridge subscribes to `application/#` on the gateway's broker, decodes the base64 LoRaWAN payload, and writes measurements to InfluxDB with tags for `node` (lora1/lora2), `dev_eui`, and `sensor`. Each uplink becomes a data point with RSSI, SNR, DR, and the decoded sensor value.

Grafana queries InfluxDB using Flux and renders live panels for temperature, humidity, RSSI, SNR, and frame count. The frame count panel is particularly useful during range testing — gaps in the counter show exactly which uplinks were lost and when.

---

## The Join and Rejoin Algorithm

Both boards use OTAA (Over-The-Air Activation) with exponential backoff:

- First join attempt at startup
- Backoff starts at 16 seconds, doubles on each miss, caps at 60 seconds
- After 3 consecutive missed ACKs on confirmed uplinks → trigger rejoin
- Backoff resets to 16 seconds on successful join

The 60-second cap is deliberate. The LoRaWAN specification suggests backing off to 10 minutes after repeated failures. In field testing, that means a board can appear stuck for up to 10 minutes after a brief gateway interruption. With a 60-second cap, worst case is one missed window then back online. For a deployable sensor, that is an acceptable trade-off.

lora-1 sends every fifth uplink as confirmed — the failure counter only increments on missed ACKs from confirmed sends. lora-2 sends every uplink confirmed, making gateway loss detection nearly instantaneous.

---

## EUI Byte Order — Read the Spec Carefully

This one cost an afternoon.

The RAK gateway UI asks for DevEUI and AppEUI as hex strings displayed MSB-first. The `lorawan-device` crate stores them LSB-first in the firmware byte array. Get it wrong and the gateway never sees a valid join request.

```rust
// DevEUI: 23ce1bfeff091fac (as shown in gateway UI)
// Stored in firmware LSB-first:
const DEV_EUI: [u8; 8] = [0xac, 0x1f, 0x09, 0xff, 0xfe, 0x1b, 0xce, 0x23];

// AppKey is MSB-first — no reversal needed
const APP_KEY: [u8; 16] = [0xb7, 0x26, ...];
```

**Lesson:** Always verify byte order against the specification, not against what looks obvious. The "obvious" order is often wrong.

---

## What This System Actually Does

lora-1 sends temperature and humidity every 30 seconds. lora-2 sends a TX counter every 10 seconds. Both display live readings on their OLEDs. Both detect gateway loss and rejoin automatically. The Grafana dashboard refreshes every 10 seconds and shows the full history for both nodes.

The entire firmware for each board fits in about 400 lines of Rust. The MQTT bridge is 300 lines of Python. There is no proprietary SDK, no vendor cloud dependency, no monthly fee.

This is LoRaWAN as it was designed to be used: open protocol, self-hosted network, full control.

---

## What's Next

With the network running reliably, the next step was taking lora-2 outside and finding out how far it actually reaches. The result surprised me — not because of the distance, but because of what mattered most.

[Read the field test results →](/posts/2026/april/i-walked-921-metres-with-lorawan-node-here-is-what-the-data-showed/)

The full source is on [GitHub](https://github.com/mapfumo/lorawan), including both firmware projects, the Docker backend, and the MQTT bridge.
