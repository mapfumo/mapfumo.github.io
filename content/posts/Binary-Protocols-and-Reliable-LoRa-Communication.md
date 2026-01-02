---
title: "Binary Protocols and Reliable LoRa Communication"
date: 2026-01-01T20:14:18+10:00
draft: false
cover:
  image: /img/binary_protocol.png
  alt: "STM32F446RE NUCLEO board with SSD1306 OLED display"
  caption: "Nucleo Board"
tags: ["Rust", "Embedded Rust", "MCU", "RTIC"]
categories: ["IIoT", "Tech"]
---

The first two weeks of this project focused on getting data moving: sensors reading correctly, LoRa modules talking, and messages arriving at the other end. That phase was intentionally simple and text-based. It worked, but it was never meant to last.

This week was about moving closer to how real embedded systems behave in the field: binary data, explicit integrity checks, and predictable delivery behavior.

Human-readable messages are convenient for debugging, but they waste bandwidth and hide failure modes. Radios don't care about readability — they care about airtime, reliability, and determinism. This week's work was about aligning the system with those realities.

**The code for this stage lives here:**  
https://github.com/mapfumo/wk3-binary-protocol

## What I Wanted to Achieve

The goals for this stage were deliberately narrow:

- Replace text-based LoRa payloads with a compact binary protocol
- Add data integrity validation using CRC
- Implement explicit acknowledgment (ACK/NACK) between nodes
- Ensure the system remains simple, observable, and recoverable

This was not about maximum throughput or perfect reliability. It was about knowing when data is valid, knowing when it isn't, and reacting predictably.

## System Overview

The system consists of two STM32 Nucleo boards communicating via RYLR998 LoRa modules:

### Node 1 (Sender)

- Reads temperature and humidity from an SHT31
- Reads gas resistance from a BME680
- Packages sensor data into a binary packet
- Transmits periodically or on button press
- Waits for acknowledgment before continuing

### Node 2 (Receiver)

- Receives LoRa packets over UART
- Validates integrity using CRC
- Deserializes binary payloads
- Displays values on an OLED
- Sends ACK or NACK back to Node 1

Both nodes run under RTIC, with strict separation between interrupt context and slow operations like display updates.

## Binary Protocol Overview (ASCII)

To make the system behavior explicit, the protocol between Node 1 and Node 2 is intentionally simple and fully observable.

There are only two message types:

- Sensor data packets (Node 1 → Node 2)
- ACK / NACK packets (Node 2 → Node 1)

No implicit behavior. No hidden retries.

### Sensor Data Packet (Node 1 → Node 2)

On the wire, the LoRa module wraps everything inside its ASCII `+RCV` envelope, but the payload itself is pure binary.

```
+RCV=<addr>,<len>,<binary_payload>,<rssi>,<snr>\r\n
```

The binary payload layout is:

```
┌─────────────── SensorDataPacket ───────────────┐
│                                                 │
│  seq_num        temperature     humidity   gas  │
│  (u16)          (i16)           (u16)      (u32)│
│                                                 │
└──────────────────────┬──────────────────────────┘
                       │
                       ▼
                  CRC-16 (u16)
```

Expanded byte-wise:

```
[ seq_hi ][ seq_lo ]
[ temp_hi ][ temp_lo ]
[ hum_hi ][ hum_lo ]
[ gas_b3 ][ gas_b2 ][ gas_b1 ][ gas_b0 ]
[ crc_hi ][ crc_lo ]
```

**Key points:**

- Fixed-size, deterministic layout
- No floating-point data on the wire
- CRC covers only the data, not the envelope
- Big-endian CRC for readability in logs

**Total payload size:**

- Data: ~8–10 bytes (postcard)
- CRC: 2 bytes

### ACK / NACK Packet (Node 2 → Node 1)

ACK packets are deliberately tiny and do not include a CRC.  
At 3 bytes total, corruption is extremely unlikely, and simplicity wins.

```
┌────── AckPacket ──────┐
│                       │
│  msg_type   seq_num   │
│  (u8)       (u16)     │
│                       │
└───────────────────────┘
```

Where:

- `msg_type = 1` → ACK
- `msg_type = 2` → NACK

This tells Node 1 exactly one thing:  
"Packet X was received correctly" or "Packet X failed CRC."

Nothing more.

### End-to-End Flow

The full interaction looks like this:

```
Node 1                                Node 2
------                                ------
Read sensors
Serialize binary packet
Append CRC
Send packet  ────────────────▶  Receive bytes
                                   Validate CRC
                                   Deserialize
                                   Update display
                                   Send ACK/NACK
Wait for ACK ◀────────────────── Send response

If ACK:
  Transition to Idle

If NACK or timeout:
  Retry (up to MAX_RETRIES)
```

There is no ambiguity:

- Every packet has a sequence number
- Every successful delivery is acknowledged
- Every failure is explicit

### Why This Design

This protocol deliberately avoids complexity:

- No fragmentation
- No queues
- No dynamic allocation
- No background retransmission logic

It's not trying to be TCP.  
It's trying to be predictable, inspectable, and boring.

For sensor telemetry over LoRa, that trade-off is intentional.

## How It Was Achieved

### Binary Data Representation

Sensor readings are no longer sent as strings. Instead, both nodes share the same binary structure:

```rust
pub struct SensorDataPacket {
    pub seq_num: u16,
    pub temperature: i16,
    pub humidity: u16,
    pub gas_resistance: u32,
}
```

Values are scaled into integers before transmission:

- Temperature → tenths of a degree
- Humidity → hundredths of a percent

This avoids floating-point data on the wire and keeps serialization predictable. The packet is serialized using `postcard`, which works well in `no_std` environments and produces minimal output.

The result is a ~10-byte payload including CRC, compared to ~25 bytes previously used by text.

### Data Integrity with CRC-16

Binary data only works if corruption is detectable.

Each packet sent by Node 1 appends a CRC-16 checksum calculated over the serialized payload. On Node 2, the CRC is recalculated and compared before any attempt to deserialize.

If the CRC does not match:

- The packet is discarded
- A NACK is sent back
- No undefined behavior occurs

This explicit validation is cheap (2 bytes) and dramatically improves robustness.

### Reliable Delivery Using a State Machine

Node 1 implements a small transmission state machine:

```rust
pub enum TxState {
    Idle,
    WaitingForAck {
        seq_num: u16,
        timeout_counter: u32,
        retry_count: u8,
    },
}
```

The logic is intentionally simple:

1. Send packet
2. Enter `WaitingForAck`
3. Wait up to 2 seconds
4. Retry up to 3 times
5. Give up and return to `Idle`

There is no queue and no buffering. This system prioritizes fresh data over guaranteed delivery. If a packet is lost, the next one replaces it.

This behavior matches the reality of sensor telemetry.

### UART Handling: Keep Interrupts Simple

One of the more important lessons this week came from UART handling on Node 2.

Earlier versions attempted to do too much inside the UART interrupt: checking flags, logging extensively, and touching shared resources. That led to data corruption and unpredictable behavior.

The final version follows a strict rule:

- UART interrupt only collects bytes
- Parsing happens outside the UART lock
- Display updates never occur in UART context

This change alone made the system reliable.

The UART handler simply drains bytes into a buffer and sets a flag when a full message (`\n`) is received. Everything else happens later, in a controlled context.

### Parsing Mixed Binary + Text Frames

The RYLR998 module wraps binary payloads inside an ASCII envelope:

```
+RCV=<addr>,<len>,<binary_data>,<rssi>,<snr>\r\n
```

Parsing this requires careful boundary handling:

1. Identify the payload length from ASCII fields
2. Extract exactly that many bytes of binary data
3. Separate payload and CRC
4. Validate CRC
5. Deserialize payload
6. Parse RSSI/SNR from trailing text

This is not pretty, but it is explicit and reliable. Importantly, binary data is never treated as UTF-8, avoiding a common mistake.

## Results

At the end of Week 3, the system now:

- Transmits compact binary packets
- Detects corrupted data reliably
- Confirms successful delivery explicitly
- Retries failed transmissions predictably
- Remains responsive and debuggable

Payload size has been reduced by roughly 60%, and failure modes are now visible instead of silent.

Most importantly, the system behaves the same way every time — which is the real goal.

## Observations

A few things stood out during this work:

- Binary protocols are simpler once you stop trying to make them human-readable
- CRCs are non-negotiable for radio links
- Interrupt handlers must be boring to be reliable
- State machines clarify behavior better than ad-hoc flags

None of this is surprising in hindsight, but it only becomes obvious once you build it, break it, and fix it.

This is exactly why I prefer learning by doing instead of following tutorials. Tutorials show the happy path. Real systems spend most of their time off it.

## Next Steps

With serialization, integrity, and acknowledgments in place, the foundation is solid.

Next steps will focus on:

- Performance measurements
- Edge-case behavior
- Tightening timing and resource usage
- Preparing for gateway integration

No new features — just making the existing ones boringly reliable.
