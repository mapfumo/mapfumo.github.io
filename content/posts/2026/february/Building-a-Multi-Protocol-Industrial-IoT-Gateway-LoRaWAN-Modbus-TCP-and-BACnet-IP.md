---
title: "Building a Multi-Protocol Industrial IoT Gateway: LoRaWAN, Modbus TCP, and BACnet/IP"
date: 2026-02-01T17:01:44+10:00
draft: false
cover:
    image: /img/bacnet_image.svg
    alt: "Multi-Protocol Industrial IoT Gateway architecture diagram"
    caption: "LoRaWAN, Modbus TCP, and BACnet/IP unified gateway"
    tags:
    [
        "Rust",
        "Embedded Rust",
        "Embassy",
        "lora",
        "lorawan",
        "influxdb",
        "Grafana",
        "STM32",
        "STM32WL55",
        "STM32F446RE",
        "modbus",
        "modbustcp",
        "bacnet",
        "docker",
        "industrialiot",
        "IIoT",
    ]
    categories: ["IIoT", "Tech"]
---

When industrial facilities need to monitor equipment and environmental conditions, they often face a challenge: different systems speak different languages. A temperature sensor in one part of the building might use Modbus TCP, while the HVAC system uses BACnet/IP, and wireless sensors communicate via LoRaWAN. Getting all this data into one place typically requires expensive proprietary gateways or complex integration projects.

I built a solution that brings these three protocols together on a single platform, using open-source tools and commodity hardware.

## The Problem

Industrial monitoring systems rarely use just one protocol. A typical facility might have:

- **Legacy equipment** using Modbus TCP (an industrial standard from the 1970s, still widely deployed)
- **Building automation systems** using BACnet/IP (the standard for HVAC and building controls)
- **Wireless sensors** using LoRaWAN (low-power, long-range radio communication)

Each protocol has its strengths. Modbus TCP is simple and reliable. BACnet/IP is designed specifically for building automation with rich device models. LoRaWAN enables battery-powered sensors that can transmit for years without maintenance.

The challenge is integration. Commercial gateways that support multiple protocols often cost thousands of dollars and lock you into proprietary ecosystems.

## The Solution

I developed a unified monitoring platform that combines all three protocols into a single dashboard. The system uses:

- **Embedded firmware** written in Rust running on STM32 microcontrollers
- **Protocol bridge software** in Python to collect data from all three networks
- **Time-series database** (InfluxDB) to store sensor readings
- **Visualization platform** (Grafana) for real-time dashboards

The entire stack is open-source and runs on standard hardware.

## System Architecture

The platform consists of four sensor nodes:

**LoRaWAN nodes (2):**

- STM32WL55 microcontrollers with integrated LoRa radio
- BME680 environmental sensors (temperature, humidity, pressure, air quality)
- Battery-powered for wireless deployment
- 30-second update intervals over multi-kilometer range

**Modbus TCP node:**

- STM32F446RE microcontroller with W5500 Ethernet controller
- SHT3x temperature/humidity sensor
- Wired Ethernet for industrial reliability
- IEEE 754 float32 register encoding

**BACnet/IP gateway:**

- Same STM32F446RE + W5500 hardware as Modbus node
- Implements BACnet Device object model (Analog Inputs for sensors, Analog Value for uptime)
- Responds to Who-Is/I-Am discovery and ReadProperty requests
- Dual-protocol: serves both Modbus TCP and BACnet/IP simultaneously

All sensor data flows into a unified InfluxDB time-series database, where Grafana visualizes it in real-time.

## Technical Implementation

### Firmware: Embassy Async Framework

The embedded firmware uses Embassy, an async/await framework for Rust. This choice was deliberate. While RTIC (Real-Time Interrupt-driven Concurrency) would provide deterministic timing, BACnet/IP and Modbus TCP run over Ethernet with millisecond-scale latencies. Network timing dominates, not MCU scheduling.

Embassy's async model made it straightforward to manage multiple concurrent tasks:

- Sensor reading every 2 seconds
- Network request handling (BACnet and Modbus on different sockets)
- OLED display updates

For protocols requiring microsecond timing—like BACnet MS/TP over RS-485—RTIC would be more appropriate. But for IP-based protocols, Embassy's ergonomics outweigh RTIC's determinism.

### Protocol Implementation: Pure Socket Programming

Rather than using existing protocol libraries, I implemented the wire formats directly. This was an educational choice—understanding how data actually moves across the network matters when troubleshooting industrial systems.

**Modbus TCP:**

- MBAP header (transaction ID, protocol ID, length, unit ID)
- Function code 0x03 (Read Holding Registers)
- IEEE 754 encoding for floating-point sensor values

**BACnet/IP:**

- BVLC (BACnet Virtual Link Control) for UDP framing
- NPDU (Network Protocol Data Unit) for routing
- APDU (Application Protocol Data Unit) for services
- Object model: Device (1234), Analog Inputs (0, 1), Analog Value (0)

**LoRaWAN:**

- OTAA (Over-The-Air Activation) for secure joining
- Encrypted payload: packed binary structs for efficiency
- RAK7268V2 WisGate Edge Lite 2 gateway (AU915)
- Gateway publishes to MQTT, Python bridge writes to InfluxDB

### Data Pipeline

Each protocol has its own Python bridge service:

```
LoRaWAN Nodes → RAK7268V2 → MQTT → mqtt_to_influx.py → InfluxDB
Modbus Nodes → modbus_to_influx.py → InfluxDB
BACnet Gateway → bacnet_to_influx.py → InfluxDB
```

All three write to the same InfluxDB bucket with protocol-specific tags, enabling unified queries across heterogeneous sensors.

The entire infrastructure runs in Docker containers, started with a single command:

```bash
./start_services.sh
```

## Practical Results

The system provides:

**Unified visibility:** One dashboard shows data from all sensors regardless of protocol
**Protocol translation:** BACnet clients can read from sensors that natively speak Modbus
**Reliability:** Each protocol uses its appropriate physical layer (wireless for flexibility, wired for mission-critical)
**Extensibility:** Adding new sensors just means deploying firmware and tagging data

## Lessons Learned

**1. Protocol choice matters less than network design**

I spent considerable time optimizing BACnet encoding, but the limiting factor is always network latency. A well-structured Modbus implementation performs identically to BACnet for simple sensor reading.

**2. Async frameworks simplify concurrent I/O**

Embassy's async model made it trivial to handle multiple network sockets and I2C sensors. Traditional embedded programming would require complex state machines or RTOS tasks.

**3. Time-series databases are essential**

Industrial monitoring generates enormous data volume. InfluxDB's tag-based indexing makes it fast to query "all temperature sensors in building A" across mixed protocols.

**4. Documentation is harder than code**

The firmware works reliably, but making it understandable to others required more effort than writing it. Clear diagrams, troubleshooting guides, and architectural decisions (like NOTES.md explaining why Embassy over RTIC) make the difference between a personal project and something others can use.

## What's Next

This project demonstrates protocol integration is achievable without proprietary hardware. The next step is robustness: industrial deployments need watchdog timers, firmware updates over the network, and comprehensive error handling.

For building automation specifically, adding support for BACnet trends and schedules would enable more sophisticated control logic. The hardware already supports this—it's a software enhancement.

## Code Deep Dive: BACnet/IP Implementation

To make this concrete, let's examine how BACnet/IP works at the wire level. The protocol has three layers, each serving a specific purpose.

### Layer 1: BVLC (BACnet Virtual Link Control)

BVLC is the "envelope" that wraps BACnet messages for UDP/IP transport. It's always 4 bytes:

```rust
/// BVLC Header (4 bytes)
pub struct BvlcHeader {
    pub bvlc_type: u8,    // Always 0x81 for BACnet/IP
    pub function: u8,     // Unicast (0x0A) or Broadcast (0x0B)
    pub length: u16,      // Total packet length
}

impl BvlcHeader {
    pub fn to_bytes(&self, buffer: &mut [u8]) -> usize {
        buffer[0] = self.bvlc_type;
        buffer[1] = self.function;
        buffer[2] = (self.length >> 8) as u8;
        buffer[3] = (self.length & 0xFF) as u8;
        4
    }
}
```

### Layer 2: NPDU (Network Protocol Data Unit)

NPDU handles routing between BACnet networks. For simple setups, it's just 2 bytes:

```rust
pub struct NpduHeader {
    pub version: u8,   // Always 0x01
    pub control: u8,   // 0x00 for no routing
}
```

### Layer 3: APDU (Application Protocol Data Unit)

The APDU contains the actual BACnet command. Here's how we respond to a ReadProperty request for a temperature sensor:

```rust
pub fn build_read_property_ack(
    buffer: &mut [u8],
    request: &ReadPropertyRequest,
    sensor_data: &BacnetSensorData,
) -> usize {
    let mut pos = 0;

    // BVLC Header
    buffer[pos] = 0x81;  // BACnet/IP
    buffer[pos + 1] = 0x0A;  // Unicast
    pos = 4;  // Fill length later

    // NPDU
    buffer[pos] = 0x01;  // Version 1
    buffer[pos + 1] = 0x00;  // No routing
    pos += 2;

    // APDU: Complex ACK
    buffer[pos] = 0x30;  // Complex ACK PDU type
    buffer[pos + 1] = request.invoke_id;  // Echo invoke ID
    buffer[pos + 2] = 0x0C;  // ReadProperty service
    pos += 3;

    // Echo the object identifier
    buffer[pos] = 0x0C;  // Context tag 0, length 4
    let object_id = ((request.object_type as u32) << 22)
                  | request.object_instance;
    buffer[pos + 1..pos + 5].copy_from_slice(&object_id.to_be_bytes());
    pos += 5;

    // Echo the property identifier
    buffer[pos] = 0x19;  // Context tag 1, length 1
    buffer[pos + 1] = request.property_id as u8;
    pos += 2;

    // Opening tag for property value
    buffer[pos] = 0x3E;
    pos += 1;

    // Encode the actual sensor value (IEEE 754 float32)
    let value = match request.object_instance {
        0 => sensor_data.temperature,
        1 => sensor_data.humidity,
        _ => 0.0,
    };
    buffer[pos] = 0x44;  // Application tag: REAL
    buffer[pos + 1..pos + 5].copy_from_slice(&value.to_be_bytes());
    pos += 5;

    // Closing tag
    buffer[pos] = 0x3F;
    pos += 1;

    // Fill in BVLC total length
    let total_len = pos as u16;
    buffer[2] = (total_len >> 8) as u8;
    buffer[3] = (total_len & 0xFF) as u8;

    pos
}
```

### Async Architecture with Embassy

The firmware runs both protocols simultaneously using Embassy's async/await model:

```rust
loop {
    Timer::after_millis(100).await;  // 10Hz main loop

    // Task A: Handle Modbus TCP (Socket 0)
    match tcp_get_status(&mut spi, &mut cs).await {
        Ok(SOCK_STATUS_ESTABLISHED) => {
            if let Ok(bytes) = tcp_recv(&mut spi, &mut cs, &mut rx_buf).await {
                let response = modbus::handle_request(
                    &rx_buf[..bytes],
                    &sensor_data,
                    &mut tx_buf
                );
                tcp_send(&mut spi, &mut cs, &tx_buf[..len]).await;
            }
        }
        _ => {}
    }

    // Task B: Handle BACnet/IP (Socket 1, UDP)
    if let Ok(bytes) = udp_rx_available(&mut spi, &mut cs).await {
        if let Ok((len, src_ip, src_port)) =
            udp_recv(&mut spi, &mut cs, &mut udp_buf).await {

            let response = bacnet::handle_packet(
                &udp_buf[..len],
                src_ip,
                src_port,
                &sensor_data,
                &mut tx_buf
            );

            match response {
                BacnetResponse::IAm(len) => {
                    // Broadcast I-Am response
                    udp_send_broadcast(&mut spi, &mut cs,
                                     BACNET_PORT, &tx_buf[..len]).await;
                }
                BacnetResponse::ReadPropertyAck { length, dest_ip, dest_port } => {
                    // Unicast ReadProperty response
                    udp_send(&mut spi, &mut cs,
                           dest_ip, dest_port, &tx_buf[..length]).await;
                }
                _ => {}
            }
        }
    }

    // Task C: Read sensor every 2 seconds
    if loop_counter % 20 == 0 {
        (temp, hum) = read_sht3x(&mut sensor).await;
        sensor_data.temperature = temp;
        sensor_data.humidity = hum;
        sensor_data.uptime += 2;
    }
}
```

The key insight: Embassy makes it trivial to manage two network protocols plus sensor I/O in a single event loop. No RTOS needed, no complex state machines—just async/await.

## Why Pure Socket Implementation?

You might notice we're building BACnet packets byte-by-byte rather than using an existing library. This was deliberate:

**Learning:** Understanding how data moves across the wire is essential when troubleshooting industrial systems. When a BACnet client can't discover your device, you need to know whether the issue is in the BVLC header, NPDU routing, or APDU encoding.

**Debugging:** With logic analyzer captures showing actual bytes, you can compare against your code to find mismatches. Library abstractions hide this.

**Constraints:** Many BACnet libraries assume a full operating system with threading and dynamic allocation. Our STM32 has 128KB RAM and no OS.

**Portfolio value:** Implementing a protocol from specification demonstrates deeper understanding than `pip install bacpypes`.

That said, for production systems, well-tested libraries (BACpypes, bacnet-stack-c) would be more appropriate. This project prioritizes education over robustness.

## Code and Resources

The complete project is open-source:

- **GitHub:** [github.com/mapfumo/bacnet-gateway](https://github.com/mapfumo/bacnet-gateway)
- **Documentation:** README with quick-start guide, protocol reference, troubleshooting
- **Firmware:** Rust with Embassy async framework
- **Stack:** Docker, InfluxDB, Grafana, Python

**Hardware used:**

- NUCLEO-F446RE boards with W5500 Ethernet modules (wired nodes)
- NUCLEO-WL55JC boards with integrated LoRa radio (wireless nodes)
- RAK7268V2 WisGate Edge Lite 2 LoRaWAN gateway (AU915)
- SHT3x and BME680 environmental sensors
- SSD1306 OLED displays

The project evolved from an earlier LoRaWAN + Modbus system by adding BACnet/IP support and consolidating all firmware into a single repository. It demonstrates that industrial protocols, while complex, are accessible to engineers willing to read specifications and implement wire formats directly.

---
