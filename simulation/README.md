# Simulation

All simulations were created in **NI Multisim 14**. The `.ms14` files are in the root of the repository.

---

## Simulation Files

| File | Stage Simulated | Key Measurement |
|------|----------------|-----------------|
| [`PowerSupply.ms14`](../PowerSupply.ms14) | Full circuit (transformer → rectifier → both buck converters) | 5V and 3.3V regulated outputs |
| [`transformer_220v_9v.ms14`](../transformer_220v_9v.ms14) | Transformer stage only | 220V AC → 9V AC step-down |
| [`220v_to_3v3_short_ckt_protection.ms14`](../220v_to_3v3_short_ckt_protection.ms14) | Short circuit protection on 3.3V rail | Current limiting behavior |

---

## How to Open

1. Install **NI Multisim 14.0** or later
2. Open any `.ms14` file directly: `File → Open`
3. Run transient analysis: `Simulate → Run` (or press **F5**)

---

## Simulation Screenshots

These were captured from the Multisim runs and are saved at the repo root:

**5V Buck Converter Output:**

![5V Simulation](../5v_simulation.jpeg)

**3.3V Buck Converter Output:**

![3.3V Simulation](../3v3_simulation.jpeg)

---

## What Was Simulated

### 1. Transformer Stage (`transformer_220v_9v.ms14`)
- Input: 220V AC, 60Hz source
- Transformer ratio: 24.44:1 (220V → 9V AC)
- Output confirmed: ~9V AC sinusoidal waveform
- Verified: No distortion, correct step-down ratio

### 2. Full Power Supply (`PowerSupply.ms14`)
- Bridge rectifier (SR560 Schottky model) + 10,000 µF filter
- Two LM2596 buck converter stages with 33 µH inductors
- Feedback resistor networks set for 5V and 3.3V outputs
- Representative resistive loads on both rails

**Key results from simulation:**

| Test Point | Simulated Value |
|------------|----------------|
| Rectified DC output | ~16V DC |
| 5V buck output | **5.02V** |
| 3.3V buck output | **3.31V** |
| Ripple (3.3V rail) | **< 30 mV** |

### 3. Short Circuit Protection (`220v_to_3v3_short_ckt_protection.ms14`)
- Simulates a short circuit event on the 3.3V output rail
- Validates that the LM2596's internal current limiting activates
- Confirms no damage to upstream components during a fault

---

## Simulation vs Hardware Comparison

| Parameter | Simulated | Hardware Measured |
|-----------|-----------|------------------|
| 5V output | 5.02V | ~5.0V |
| 3.3V output | 3.31V | ~3.3V |
| 3.3V ripple | < 30 mV | < 30 mV |
| 5V max current | 2A (modeled load) | 2A stable |

Simulation results closely matched hardware measurements. Minor deviations are expected due to ideal component models in Multisim (no PCB parasitics, ideal transformer coupling).

---

## Notes

- The LM2596 model in Multisim uses `LM2596T-5.0/LF03` and `LM2596T-3.3/LF03` (fixed voltage variants) as a behavioral reference. The ADJ variant's behavior matches when the feedback network is configured correctly per [`../docs/04-calculations.md`](../docs/04-calculations.md).
- `1N5824` / `1N5825` Schottky models are used as SR560 equivalents — electrically identical for simulation purposes.
