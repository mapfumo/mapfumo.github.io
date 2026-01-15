---
title: "About"
Description: 'Aut inveniam viam aut faciam - "I shall either find a way or make one"'
layout:
---

---

![Antony Mapfumo](/img/ants.jpg#floatleft "Antony Mapfumo")

I'm Tony — a network engineer transitioning into Industrial IoT and embedded systems engineering. Currently building production-grade IIoT systems in Embedded Rust — LoRaWAN sensor networks, Modbus TCP slaves, real-time data pipelines, and unified monitoring platforms.

**Recent Work:**

- **Unified IIoT Platform**: 4-node system combining LoRaWAN + Modbus TCP → single Grafana dashboard. Includes STRIDE security analysis identifying 12+ threats with prioritized mitigations.
- **LoRaWAN Migration**: Transitioned from RYLR998 modules to native STM32WL55 radio with dual-node network (SHT41, BME680 sensors).
- **Modbus TCP Slaves**: STM32F446 + W5500 Ethernet implementing IEEE 754 float32 register encoding, concurrent client handling.
- **MQTT Pipelines**: LoRa → MQTT → InfluxDB → Grafana with sub-second end-to-end latency.

---

**Technical Stack:**

- **Embedded:** Rust (Embassy async HAL, RTIC), C++, STM32 (F4/WL55)
- **Protocols:** LoRaWAN, Modbus TCP, MQTT, OPC-UA, binary formats (Postcard)
- **Tools:** probe-rs, logic analyzers, Docker, InfluxDB, Grafana
- **Security:** STRIDE threat modeling, IEC 62443 awareness

**Engineering Philosophy:** Systematic, hardware-validated, production-minded. Every project includes: working hardware, comprehensive documentation, security analysis, and open-source code.

## Portfolio

- **GitHub:** [github.com/mapfumo](https://github.com/mapfumo)
