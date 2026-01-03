---
title: "Building Deterministic IIoT Systems With Embedded Rust and RTIC"
date: 2025-12-02T17:53:41+10:00
draft: false
cover:
  image: /img/rtic_img.png
  alt: "STM32F446RE NUCLEO board with SSD1306 OLED display"
  caption: "Nucleo Board"
tags: ["Rust", "Embedded Rust", "MCU", "RTIC"]
categories: ["IIoT", "Tech"]
---

In the world of Industrial IoT (IIoT), reliability is everything. Sensors must read accurately, actuators must respond predictably, and communication must flow without surprises. On the embedded side, this means precise control of hardware resources, minimal overhead, and code that behaves deterministically. Enter Embedded Rust and RTIC—a pairing that gives engineers both safety and performance without compromise.

---

## Why Embedded Rust?

Rust’s focus on memory safety, zero-cost abstractions, and strict compile-time checks makes it ideal for embedded systems. Unlike C, Rust prevents **entire classes of bugs** like null pointer dereferences or buffer overflows. When your MCU has just a few kilobytes of RAM, these protections are not optional—they’re essential.

Embedded Rust isn’t just safer—it’s also **fast and predictable.** With careful hardware abstractions, you can write code that is both high-level and close to the metal.

---

## Introducing RTIC: Real-Time Interrupt-driven Concurrency

RTIC, or **Real-Time Interrupt-driven Concurrency**, is a framework built for embedded Rust that makes deterministic scheduling easy. It’s designed for hard real-time tasks, where timing guarantees are critical.

RTIC works by:

- Mapping hardware interrupts to tasks
- Managing shared resources safely across interrupts
- Providing static scheduling analysis at compile time

Unlike traditional RTOS systems, RTIC avoids dynamic scheduling and preemptive threads, reducing overhead and unpredictability. It’s particularly well-suited for MCUs with constrained RAM or real-time control loops.

**Key benefits:**

- **Deterministic execution:** Know exactly when tasks will run
- **Safe concurrency:** Rust enforces access rules at compile time
- **Minimal footprint:** No dynamic memory allocation or garbage collection

Here’s a simplified workflow of RTIC:
![RTIC Workflow](/img/rtic_img_2.png)

---

## The Hardware Project

To bring RTIC to life, we built a minimal but fully functional demo:

- **MCU:** STM32 (any Cortex-M board works)
- **LED:** Onboard, serves as a visual heartbeat
- **OLED Display:** 128x64, connected via I²C, shows task status
- **Timer:** Triggers periodic events every 500ms to update tasks

This project’s main goal is to demonstrate **deterministic behavior and sensor-to-display communication**, while laying the groundwork for more advanced iterations—like adding **LoRa modules** for long-range wireless communication. Think of it as a stepping stone for future IIoT experiments.

---

## How the Code Works

RTIC makes the code clean and predictable:

1. **Peripheral Setup:** In init(), GPIO, I²C, and timers are configured
2. **Task Scheduling:** RTIC schedules tim2_handler() on timer interrupts
3. **Task Execution:** Each tick, the handler toggles the LED, updates the OLED, and logs data

```rust
#[rtic::app(device = stm32f4xx_hal::pac, peripherals = true)]
mod app {
    #[shared]
    struct Shared {}

    #[local]
    struct Local {
        led: LedPin,
        oled: OledDisplay,
    }

    #[init]
    fn init(ctx: init::Context) -> (Shared, Local, init::Monotonics) {
        let led = setup_led();
        let oled = setup_oled();
        (Shared {}, Local { led, oled }, init::Monotonics())
    }

    #[task(binds = TIM2, local = [led, oled])]
    fn tim2_handler(ctx: tim2_handler::Context) {
        ctx.local.led.toggle();
        ctx.local.oled.update("Tick");
    }
}

```

This structure scales naturally: replace the LED with a sensor, the OLED with a cloud publisher, and later add LoRa or other wireless modules. RTIC ensures each component behaves predictably, even as system complexity grows.

**[GitHub: STM32 RTIC IIoT Demo](https://github.com/mapfumo/rtic_oled_bringup)**

---

## From Demo to IIoT

Once the basic RTIC setup is stable, it becomes a foundation for IIoT applications:

- \*\*Sensor Integration: Swap the LED for a temperature or vibration sensor
- \*\*Data Logging: Send updates to a cloud server via MQTT or HTTP
- \*\*Wireless Communication: Integrate LoRa modules to bridge remote devices
- \*\* Deterministic Control Loops: Guarantee actuator response within tight timing constraints

By building incrementally on this simple demo, engineers can maintain **full control over timing, concurrency, and hardware interactions,** all without sacrificing safety or clarity.

---

## Why Not Other RTOS Frameworks?

You might wonder why not use FreeRTOS or Embassy. A brief comparison:

- **FreeRTOS:** Widely used, mature, but dynamic and preemptive; requires careful handling to avoid subtle bugs
- **Embassy:** Async/await model for embedded Rust; excellent for IO-heavy tasks, but slightly higher learning curve and less deterministic for real-time loops
- **RTIC:** Static scheduling, predictable, minimal footprint—perfect for real-time deterministic IIoT control

For our purposes—demonstrating hardware determinism and building a modular foundation—RTIC hits the sweet spot.

---

## Exploring Further: LoRa and Beyond

One of the exciting directions is LoRa integration. Because our demo already uses a timer-based scheduler and structured peripheral handling, adding a LoRa module is straightforward:

- \*\*Periodically collect sensor data
- \*\*Queue for LoRa transmission
- \*\*Schedule retransmissions deterministically if needed

This approach scales: **each new module or sensor fits naturally into the RTIC task model**, preserving predictability and reliability.

---

## Open Source and Learning

All code and setup details are available on **[GitHub: STM32 RTIC IIoT Demo](https://github.com/mapfumo/rtic_oled_bringup)**
.

Studying and modifying this project is a great way to:

- Understand **real-time scheduling**
- Practice **embedded Rust**
- Learn how **IIoT nodes** can scale from simple LEDs to LoRa-enabled sensor networks

---

## Conclusion

Embedded Rust + RTIC offers a modern, deterministic approach to IIoT development. By starting small—an LED, a timer, an OLED—we create a robust foundation for sensors, actuators, and wireless communication.

The beauty of this approach is **scalability without losing control**. Every new component—LoRa, cloud integration, more sensors—fits naturally into the RTIC framework.

For engineers who value safety, predictability, and minimal overhead, this combination is a compelling way to build **industrial-grade IoT devices**, right from your desk.
