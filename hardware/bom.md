# Bill of Materials (BOM)

All components are available in Indian electronics markets (local or online via Robu, Rajiv, Amazon).

**Total Estimated Cost: ₹583**

---

## Component List

| # | Component | Specification | Qty | Unit Cost (₹) | Total (₹) | Where Used |
|---|-----------|---------------|-----|--------------|-----------|------------|
| 1 | Step-Down Transformer | 230V AC → 12V AC, 2A | 1 | 250 | 250 | AC isolation + step-down |
| 2 | SR560 Schottky Diode | 5A, 60V | 4 | 15 | 60 | Bridge rectifier |
| 3 | LM2596-ADJ Buck Converter IC | 3A, 150kHz | 2 | 35 | 70 | 5V and 3.3V regulation |
| 4 | Electrolytic Capacitor | 10,000 µF / 35V | 1 | 70 | 70 | Bulk filter (rectifier output) |
| 5 | Electrolytic Capacitor | 470 µF / 25V | 2 | 10 | 20 | Buck converter input filter |
| 6 | Electrolytic Capacitor | 10 µF / 25V | 2 | 5 | 10 | Output smoothing |
| 7 | Electrolytic Capacitor | 47 µF / 25V | 2 | 5 | 10 | Output smoothing |
| 8 | Inductor | 33 µH / 5A | 1 | 43 | 43 | Buck converter energy storage |
| 9 | Resistors (various) | 1.1kΩ, 3.3kΩ, 1.5kΩ, 2.5kΩ | Several | ~1 each | 10 | Feedback voltage dividers |
| 10 | Dotted PCB Board | ~10cm × 8cm | 1 | 40 | 40 | Prototype assembly |
| 11 | Fuse + Holder | 2A, 250V | 1 | 20 | 20 | AC input protection |
| **Total** | | | | | **₹603*** | |

*Actual BOM assembled for ~₹583 with local pricing.*

---

## Key Component Notes

### LM2596-ADJ (×2)
- Handles 3A continuous with minimal heatsinking
- Adjustable output via feedback resistor network
- Built-in thermal shutdown and current limiting
- Available: Robu.in, local electronics shops
- **Do not substitute with LM2596-5.0 fixed version** — the ADJ variant is needed to set exact voltages

### SR560 Schottky Diode (×4)
- Used in bridge rectifier configuration (×4 for full-bridge)
- Also used as freewheeling diode in each buck converter stage
- Lower Vf (0.5V) vs 1N4007 (1.1V) → less heat at 2A
- Must be Schottky for 150 kHz compatibility in freewheeling role

### 33 µH / 5A Inductor
- Critical: current rating must be **≥ 5A** to prevent saturation
- Common mistake: using a 33 µH / 1A or 2A inductor — will saturate and fail
- Check: inductor should not get hot at 2A load; if it does, check saturation current spec

### 10,000 µF / 35V Capacitor
- Physically large — plan PCB space accordingly
- **Must be rated ≥ 25V**; 35V provides safety margin
- ESR should be low (< 0.1Ω); standard electrolytic is fine at 50Hz ripple frequency

---

## Where to Buy (India)

| Source | Suitable For |
|--------|-------------|
| Robu.in | LM2596, inductors, Schottky diodes |
| Rajiv Components (Bangalore) | Transformers, capacitors, resistors |
| Amazon India | Most ICs and passives |
| Local electronics market | Transformers, PCB boards, fuses |
