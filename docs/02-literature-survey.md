# Literature Survey

## Background

IoT automation platforms like the ESP32 require stable voltage with minimal noise for reliable wireless communication and accurate sensor readings. Poor power supply design is a leading cause of automation system failure. This survey reviews existing topologies and identifies the best fit for ESP32-based applications.

---

## Power Supply Topologies

### Linear Power Supplies

- Step down AC via transformer → rectify → filter → linear regulator
- **Pros:** Very low ripple, simple design, low noise
- **Cons:** Low efficiency (40–50% typical); excess voltage dropped as heat; bulky at high current
- **Verdict for IoT:** Unsuitable at 2A+ loads — too much heat, too little efficiency

### Switched Mode Power Supplies (SMPS)

- Use high-frequency switching to transfer energy efficiently
- **Pros:** 80–92% efficiency, compact, low thermal output
- **Cons:** Switching action introduces ripple and EMI; requires careful filtering
- **Verdict for IoT:** The right category — proper filtering makes it suitable

### Buck Converters

A specific SMPS topology for **step-down** conversion:
- Inductor stores energy during switch ON, releases during switch OFF
- Control loop maintains regulated output
- LM2596 series: popular in embedded/IoT for high efficiency and 3A current capability
- **Key design requirement:** Proper inductor and output capacitor selection to minimize ripple

### Linear Regulators (LDO)

- Regulate by dissipating excess voltage as heat
- Examples: 7805, AMS1117, LD1117
- **Pros:** Zero switching noise, simple circuit
- **Cons:** Efficiency depends on (Vout/Vin) ratio — poor at high input voltages; can't handle high current without large heatsinks
- **Verdict:** Useful as a post-regulator on a low-voltage rail; not as a primary 230V→5V stage

---

## ESP32 Power Requirements

The ESP32 is a dual-core MCU with integrated WiFi and Bluetooth. Key power characteristics:

| Parameter | Value |
|-----------|-------|
| Operating Voltage | 3.3V |
| Typical current (idle) | 20–80 mA |
| Peak current (WiFi TX) | 250–500 mA |
| ADC reference sensitivity | Affected by supply noise |

**Implication:** The 3.3V rail must hold steady during 250–500 mA WiFi bursts. A supply that sags under load will cause the ESP32 to reset mid-operation.

### Full System Load Estimate

| Component | Current |
|-----------|---------|
| 3× Relay Modules | ~250 mA |
| Ultrasonic Sensor (HC-SR04) | 15 mA |
| Soil Moisture Sensor | 10–20 mA |
| ESP32 WiFi Peak | 250–500 mA |
| **Total Estimated Load** | **600–800 mA** |

---

## Power Supply Requirements for IoT

### Voltage Stability
- Voltage variation must be < ±5% of nominal to avoid ESP32 brownout resets
- Transient response must recover quickly from sudden load steps (WiFi bursts)

### Noise and Ripple
- Switching supplies introduce ripple that corrupts ADC readings
- For the ESP32's 12-bit ADC: ripple should ideally be < 50 mV; < 30 mV is preferred
- Soil moisture and ultrasonic sensors are sensitive to power noise

### Efficiency
- 24/7 operation makes efficiency important for thermal management and electricity cost
- Linear regulators at 2A from a 12V input dissipate ~14W as heat — not viable

---

## Design Decision Based on Survey

The literature clearly supports a **switched-mode dual-output supply** as the best architecture:

| Criterion | Linear | LDO | Buck (LM2596) |
|-----------|--------|-----|---------------|
| Efficiency | Low | Medium | High |
| Current Handling | Medium | Low | High (3A) |
| Ripple | Very Low | Low | Medium (filterable) |
| Thermal Performance | Poor | Medium | Good |
| Design Complexity | Low | Low | Medium |
| Cost | Low | Low | Medium |

**Selected approach:** Dual LM2596-ADJ buck converter stages — one for 5V, one for 3.3V — fed from a transformer-isolated rectified supply.

---

## References

- Espressif Systems, "ESP32 Series Datasheet," v4.0, 2023
- Texas Instruments, "LM2596 SIMPLE SWITCHER 150-kHz 3-A Step-Down Voltage Regulator," SNVS124D, 2016
- M. H. Rashid, *Power Electronics: Circuits, Devices, and Applications*, 4th ed., Pearson, 2014
- Vishay, "SR560 Schottky Barrier Rectifier Datasheet," Rev. 1.1, 2018
- S. Kumar and A. R. Al-Ali, "Power Management Strategies for IoT Nodes," IEEE Access, vol. 8, 2020
- Texas Instruments, "AN-1149 Layout Guidelines for Switching Power Supplies," SNVA021C, 2013
