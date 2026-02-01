---
title: "About"
Description: 'Aut inveniam viam aut faciam - "I shall either find a way or make one"'
layout:
---

---

![Antony Mapfumo](/img/ants_bw.jpg#floatleft "Antony Mapfumo")

I'm Tony — a network engineer and embedded systems developer specializing in Industrial IoT. I build production-grade IIoT systems in Embedded Rust: multi-protocol gateways, sensor networks, and real-time data pipelines. The combination of network engineering and embedded development is a natural fit — industrial protocols are just packets with different semantics.

**Recent Work:**

- **Multi-Protocol Gateway**: Unified platform integrating LoRaWAN, Modbus TCP, and BACnet/IP on STM32 microcontrollers. Pure socket implementations, Embassy async framework, Docker-based infrastructure.
- **LoRaWAN Sensor Network**: Native STM32WL55 radio with OTAA join, BME680/SHT41 environmental sensors, RAK7268V2 gateway.
- **Modbus TCP Implementation**: STM32F446 + W5500 Ethernet with IEEE 754 float32 register encoding, concurrent client handling.
- **Data Pipelines**: Protocol bridges feeding InfluxDB time-series database with Grafana visualization. Sub-second end-to-end latency.

---

**Technical Stack:**

- **Embedded:** Rust (Embassy async, RTIC), STM32 (F4/WL55), bare-metal firmware
- **Protocols:** LoRaWAN, Modbus TCP, BACnet/IP, MQTT, OPC-UA
- **Infrastructure:** Docker, InfluxDB, Grafana, Python
- **Tools:** probe-rs, logic analyzers, Wireshark
- **Security:** STRIDE threat modeling, IEC 62443 awareness

**Engineering Philosophy:** Systematic, hardware-validated, production-minded. Every project ships with working hardware, comprehensive documentation, and open-source code.

## Portfolio

- **GitHub:** [github.com/mapfumo](https://github.com/mapfumo)
