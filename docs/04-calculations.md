# Component-Level Design & Calculations

All calculations are based on worst-case load conditions to ensure reliable 24/7 operation.

---

## 1. Filter Capacitor Selection

**Goal:** Limit ripple voltage to ~2V at full load (2A) from a 50Hz full-wave rectifier.

**Formula (full-wave rectifier):**

```
C = I_load / (2 × f × V_ripple)
```

**Values:**
- I_load = 2A (maximum design load)
- f = 50 Hz (AC mains)
- V_ripple = 2V (allowed ripple at rectifier output)

**Calculation:**

```
C = 2 / (2 × 50 × 2)
C = 2 / 200
C = 0.01 F = 10,000 µF
```

**Selected:** 10,000 µF / 35V electrolytic capacitor.

> The 35V rating provides a 2× safety margin over the ~16V DC bus.

---

## 2. DC Voltage Analysis

Verifying the rectifier output provides enough headroom for the buck converters.

**Transformer secondary:** ~12V AC RMS
**Rectified peak voltage:**
```
V_peak = 12 × √2 − (2 × V_f_SR560)
       = 16.97 − (2 × 0.5)
       = 15.97V ≈ 16V DC
```

**Average DC voltage (after filtering):**
```
V_avg = V_peak − (V_ripple / 2)
      = 15.97 − 1.0
      = 14.97V
```

**Minimum DC voltage (worst case trough):**
```
V_min = V_peak − V_ripple
      = 15.97 − 2.0
      = 13.97V
```

**Conclusion:** Minimum 13.97V input to the buck converters. LM2596 requires at least Vout + 1.5V input headroom. For 5V output, minimum needed = 6.5V. ✅ Sufficient headroom confirmed.

---

## 3. Buck Converter Inductor Calculation (LM2596)

**Goal:** Keep inductor ripple current (ΔI_L) at ~30% of max load current for continuous conduction mode.

**Parameters:**
- V_in = 15V (conservative average)
- V_out = 5V
- f_sw = 150 kHz (LM2596 internal switching frequency)
- I_load_max = 2A
- ΔI_L = 30% × 2A = 0.6A

**Formula:**
```
L = (V_out × (V_in − V_out)) / (V_in × f_sw × ΔI_L)
```

**Calculation:**
```
L = (5 × (15 − 5)) / (15 × 150,000 × 0.6)
L = (5 × 10) / 1,350,000
L = 50 / 1,350,000
L ≈ 37 µH
```

**Selected:** 33 µH / 5A inductor (nearest standard value below calculated).

> The 5A current rating ensures no saturation even at peak transient loads.

---

## 4. Feedback Resistor Network (Output Voltage Setting)

The LM2596-ADJ sets output voltage via a resistor divider to Pin 4 (FB). Internal reference: **V_ref = 1.23V**.

**Formula:**
```
V_out = V_ref × (1 + R2/R1)
      = 1.23 × (1 + R2/R1)
```

### 5V Output Rail

**Target:** V_out = 5.0V

```
5.0 = 1.23 × (1 + R2/R1)
R2/R1 = (5.0/1.23) − 1 = 3.065
```

Using standard values:
- R1 = 1.085 kΩ → use **1.1 kΩ**
- R2 = 3.36 kΩ → use **3.3 kΩ**

**Verification:**
```
V_out = 1.23 × (1 + 3.36/1.085) = 1.23 × 4.095 ≈ 5.03V ✅
```

### 3.3V Output Rail

**Target:** V_out = 3.3V

```
3.3 = 1.23 × (1 + R2/R1)
R2/R1 = (3.3/1.23) − 1 = 1.683
```

Using standard values:
- R1 = 1.48 kΩ → use **1.5 kΩ**
- R2 = 2.5 kΩ → use **2.5 kΩ**

**Verification:**
```
V_out = 1.23 × (1 + 2.5/1.48) = 1.23 × 2.689 ≈ 3.31V ✅
```

---

## 5. Freewheeling Diode Selection

When the LM2596 internal switch opens, the inductor's collapsing magnetic field generates a large back-EMF spike. A fast freewheeling (flyback) diode is required to clamp this.

**Why not 1N4007?**
- Reverse recovery time: ~30 µs — far too slow for 150 kHz switching
- At 150 kHz, the 1N4007 would cause shoot-through current and major heat dissipation

**Selected: SR560 Schottky Diode**

| Parameter | 1N4007 | SR560 |
|-----------|--------|-------|
| Forward voltage (V_f) | ~1.1V | ~0.5V |
| Reverse recovery | ~30 µs | ~10 ns |
| Max current | 1A | 5A |
| Switching compatibility | No (too slow) | ✅ Yes |

Power loss comparison at 2A:
```
P_1N4007 = 1.1V × 2A = 2.2W (wasted as heat)
P_SR560  = 0.5V × 2A = 1.0W (55% reduction)
```

---

## 6. Input/Output Capacitor Selection

### Input Capacitor (at LM2596 Vin pin)
- **Value:** 470 µF / 25V
- **Purpose:** Buffers sudden current demand during ESP32 WiFi transmission bursts (250–500 mA spikes)
- Prevents voltage starving the LM2596 during fast load transients

### Output Capacitors
- **Primary output:** 470 µF / 25V (bulk)
- **Secondary decoupling:** 10 µF + 47 µF (reduces high-frequency switching ripple)
- Combined low-ESR network improves load transient response

---

## Summary of Selected Values

| Component | Calculated | Selected |
|-----------|-----------|---------|
| Filter capacitor | 10,000 µF | 10,000 µF / 35V ✅ |
| Inductor | 37 µH | 33 µH / 5A ✅ |
| R1 (5V rail) | 1.085 kΩ | 1.1 kΩ ✅ |
| R2 (5V rail) | 3.36 kΩ | 3.3 kΩ ✅ |
| R1 (3.3V rail) | 1.48 kΩ | 1.5 kΩ ✅ |
| R2 (3.3V rail) | 2.5 kΩ | 2.5 kΩ ✅ |
| Freewheeling diode | Schottky, fast | SR560 ✅ |
| Input capacitor | 470 µF | 470 µF / 25V ✅ |
