# Results and Discussion

Validation was done in two phases: simulation first, then hardware prototype testing.

---

## Phase 1: Simulation

**Tools used:** Proteus, NI Multisim

The simulation model included:
- 230V AC source → transformer model
- SR560 bridge rectifier + 10,000 µF filter capacitor
- Two LM2596-ADJ buck converter stages
- 33 µH inductor and SR560 freewheeling diode
- Representative load resistors matching ESP32 + relay current draw

### Transformer Stage

The step-down transformer correctly reduced 220V AC to ~12V AC secondary output. Waveform confirmed proper sinusoidal shape with no distortion.

### Rectifier and Filter Output

After the SR560 bridge rectifier and 10,000 µF filter capacitor:
- Rectified DC: **~16V**
- Ripple at rectifier output: within calculated 2V budget

### Buck Converter Outputs

| Stage | Target | Simulated Output |
|-------|--------|-----------------|
| Buck #1 | 5V | **5.02V** |
| Buck #2 | 3.3V | **3.31V** |

Both outputs were stable under varying load conditions.

### Ripple Analysis

| Rail | Ripple Voltage |
|------|---------------|
| 5V rail | < 50 mV |
| 3.3V rail | **< 30 mV** ✅ |

The 33 µH inductor smoothed switching current effectively. SR560 freewheeling diode reduced conduction losses compared to standard silicon diodes.

### Simulation Summary Table

| Test Point | Observed Value |
|------------|---------------|
| Transformer secondary | ~14.1V AC |
| Rectified output | ~16V DC |
| Buck #1 output | 5.02V DC |
| Buck #2 output | 3.31V DC |
| Ripple (3.3V rail) | < 30 mV |

---

## Phase 2: Hardware Prototype

The prototype was built on a dotted PCB board. Components soldered manually.

### Setup

- Step-down transformer (230V → 12V AC)
- SR560 Schottky bridge rectifier
- 10,000 µF / 35V filter capacitor
- Two LM2596-ADJ buck converter stages
- 33 µH / 5A inductor
- Feedback resistor networks (5V and 3.3V configurations)
- Additional output capacitors: 470 µF, 10 µF, 47 µF

### Measured Results

| Parameter | Measured |
|-----------|---------|
| 5V rail output | ~5.0V |
| 5V rail max current | 2A stable |
| 3.3V rail output | ~3.3V |
| 3.3V rail max current | 1A stable |
| Ripple (3.3V rail) | < 30 mV |
| ESP32 WiFi peak handled | 250–500 mA without voltage drop |
| Thermal behavior | No overheating at continuous load |
| Controller resets during WiFi TX | None observed |

### Observations

- The 10,000 µF bulk capacitor effectively buffered load transients from ESP32 WiFi bursts
- SR560 diodes ran cooler than equivalent 1N4007 diodes would have at 2A (lower Vf)
- Both buck converter outputs remained stable within ±50 mV under varying load
- No heatsink required on LM2596 at tested loads (thermal shutdown not triggered)
- System ran continuously without failures during testing

---

## Discussion

### Did simulation match hardware?

Yes, within acceptable margins. Simulated ripple (<30 mV) matched hardware measurements. Voltage outputs were within ±1% of target.

### Efficiency comparison

At 2A output, 5V rail:

| Regulator | Efficiency | Heat at 2A |
|-----------|-----------|------------|
| IC 7805 (linear) | ~45% | ~14W |
| LM2596 (buck) | ~88–90% | ~1.5W |

The LM2596 choice eliminated the need for heatsinking and made continuous operation practical.

### Key takeaway

A well-filtered switching supply with a dedicated 3.3V rail delivers clean, stable power that off-the-shelf adapters cannot. The ESP32 ran without a single reset during WiFi-heavy testing — the primary failure mode of cheap adapter builds.
