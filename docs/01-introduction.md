# Introduction

## The Problem

Modern IoT-based automation systems — smart water tanks, irrigation controllers, home monitoring — rely on microcontrollers like the ESP32 running **24/7**. The most overlooked failure point in these systems isn't the software or the sensors. It's the power supply.

### Where Normal Adapters Fail

Most automation builds use whatever USB adapter is lying around. These fail in practice because:

- **Single voltage only** — ESP32 needs 3.3V; relays need 5V. You can't run both cleanly from one unregulated rail.
- **High ripple & noise** — Cheap adapters introduce electrical noise that corrupts ultrasonic and soil moisture sensor readings.
- **Insufficient current headroom** — ESP32 peaks at 250–500 mA during WiFi transmission. Under-rated adapters cause voltage sag, which causes random controller resets.
- **No galvanic isolation** — Outdoor automation (pumps, irrigation) requires safe separation from 230V mains. Most adapters skip this.

### The Core Gap

> Without a properly designed power supply, your automation system becomes unreliable — no matter how good the sensors or ESP32 code are.

---

## Motivation

Two specific use cases drove this design:

1. **Automated Water Tank Management** — Level sensing + pump control running all day. Any power glitch → pump runs dry or tank overflows. Both cause hardware damage.
2. **Smart Irrigation** — Soil moisture readings fed to a relay controlling a valve. Noisy power → wrong ADC readings → wrong watering decisions.

Both applications demand:
- Continuous, stable power (no sags, no resets)
- Separate clean rails for logic and actuators
- Safety isolation from mains

---

## Objectives

1. Design a **dual-output regulated power supply** providing stable 5V and 3.3V DC.
2. Use **LM2596-ADJ buck converter** stages for high efficiency and adequate current capacity.
3. Achieve **ripple < 30 mV** on the 3.3V logic rail to prevent sensor noise and controller resets.
4. Handle **peak ESP32 WiFi current loads** (250–500 mA) without voltage drop.
5. Ensure **galvanic isolation** from 230V AC mains via a step-down transformer.
6. Validate performance through simulation (Proteus/Multisim) and hardware prototype testing.

---

## Problem Statement

> Design and Implementation of a Regulated AC to DC Multi-Voltage Power Supply for ESP32-Based Smart Automation Systems.

---

## Scope

This project covers:
- AC mains input (230V, 50Hz) → regulated DC outputs (5V, 3.3V)
- Transformer-based isolation stage
- Full-bridge rectification and capacitor filtering
- Dual LM2596-ADJ buck converter regulation
- Validation for relay modules, ultrasonic sensors, soil moisture sensors, and ESP32

Out of scope:
- ESP32 firmware / automation logic
- PCB fabrication (prototype built on dotted PCB)
- Battery backup / solar integration (noted in future scope)

---

## Report Structure

| File | Contents |
|------|----------|
| [`02-literature-survey.md`](02-literature-survey.md) | Review of power supply topologies |
| [`03-system-design.md`](03-system-design.md) | Block diagram, alternatives, flowchart |
| [`04-calculations.md`](04-calculations.md) | Component-level design calculations |
| [`05-results.md`](05-results.md) | Simulation and hardware test results |
| [`06-future-scope.md`](06-future-scope.md) | Future improvements and societal impact |
| [`../hardware/bom.md`](../hardware/bom.md) | Bill of Materials |
| [`../hardware/component-selection.md`](../hardware/component-selection.md) | Component selection rationale |
