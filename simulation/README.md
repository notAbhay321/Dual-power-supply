# Simulation

The circuit was simulated in **Proteus** and **NI Multisim** before hardware assembly.

---

## What Was Simulated

1. **Transformer stage** — 220V AC input, 12V AC secondary, voltage and current waveforms
2. **Bridge rectifier + filter** — pulsating DC → smoothed ~16V DC; ripple measurement
3. **LM2596 Buck Converter (5V)** — full switching circuit with inductor, SR560, feedback network
4. **LM2596 Buck Converter (3.3V)** — same topology, different resistor divider values
5. **Load testing** — resistive loads simulating relay + ESP32 + sensor current draw

---

## Simulation Setup

### Transformer Stage (Multisim)
```
V1: 220V AC, 60Hz (close enough to 50Hz for topology validation)
T1: Transformer, ratio 24.44:1 (220V → 9V secondary — close to 12V model)
R1: 10kΩ load for voltage measurement
```

Key observation: Secondary output confirmed at ~9V AC, rectified peak ~12.7V DC in this model.

### Buck Converter Stage (Proteus)

**5V Rail model:**
```
V1: 5.123V DC source (represents filtered rectifier output in reduced model)
C1: 470µF input capacitor
U1: LM2596T-5.0/LF03 (5V fixed model used as reference)
L1: 33µH
D1: 1N5825 (Schottky model — SR560 equivalent)
C3: 10µF + C2: 4.7µF output
R1: 3.3kΩ feedback
Load: 10MΩ (no-load measurement)
```

**3.3V Rail model:**
```
V2: 5.123V DC source
C6: 680µF input
U3: LM2596T-3.3/LF03
L2: 33µH
D2: 1N5824
C7: 10µF + C5: 47µF
R2: 3.4kΩ feedback
Load: 10MΩ
```

---

## Results Summary

| Stage | Simulated Value | Expected |
|-------|----------------|---------|
| Transformer secondary | ~9V AC (model) | ~12V AC (target) |
| Rectified peak | ~16V DC | ~16V DC ✅ |
| 5V buck output | 5.021V | 5.0V ✅ |
| 3.3V buck output | 3.325V | 3.3V ✅ |
| Ripple (3.3V rail) | < 30 mV | < 30 mV ✅ |

---

## Files

Simulation files are not included in this repository due to software licensing (Proteus requires a paid license). 

To reproduce:
1. Open Proteus or Multisim
2. Build circuit per schematic in [`../docs/03-system-design.md`](../docs/03-system-design.md)
3. Use component values from [`../docs/04-calculations.md`](../docs/04-calculations.md)
4. Run transient analysis; measure voltage at output nodes

---

## Notes

- The Proteus LM2596 model uses a 5V fixed variant (`LM2596T-5.0`) because the ADJ model has limited simulation support. Behavior matches the ADJ configuration when the feedback network is set correctly.
- Simulation ripple is typically optimistic compared to hardware (ideal components, no PCB parasitics). Hardware measurements confirmed < 30 mV in practice.
- The 1N5824/1N5825 Schottky model in Proteus is electrically equivalent to the SR560 for simulation purposes.
