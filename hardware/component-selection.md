# Component Selection Rationale

This document explains the reasoning behind each major component choice. For calculations that supported these decisions, see [`../docs/04-calculations.md`](../docs/04-calculations.md).

---

## Voltage Regulator: LM2596-ADJ (vs. alternatives)

| Feature | IC 7805 | AMS1117-3.3 | MP1584 | **LM2596-ADJ** |
|---------|---------|-------------|--------|----------------|
| Type | Linear | LDO | Buck | Buck |
| Max current | 1A | 1A | 3A | **3A** |
| Efficiency | ~45% | ~60–70% | ~92% | **~88–90%** |
| Heat at 2A | Very high (~14W) | High | Very low | **Low (~1.5W)** |
| Package size | TO-220 (easy) | SOT-223 (small) | SOP-8 (tiny) | **TO-263 (easy)** |
| Hand-solderable | ✅ | ✅ | ⚠️ (small pads) | ✅ |
| Cost | ₹25 | ₹20 | ₹90 | **₹35** |

**Decision:** LM2596-ADJ selected over MP1584 despite marginally lower efficiency because:
1. Larger package is easier to hand-solder on dotted PCB
2. Well-documented application circuit in TI datasheet
3. Better price/performance than MP1584 for this current range
4. Built-in thermal shutdown reduces risk of damage

**IC 7805 rejected** because at 2A from 16V input, it dissipates (16-5)×2 = 22W — requires a massive heatsink, and is thermally impractical for 24/7 operation.

---

## Rectifier Diode: SR560 (vs. 1N4007)

**Why Schottky over standard silicon:**

| Property | 1N4007 | SR560 |
|----------|--------|-------|
| Forward voltage (Vf) | ~1.1V | ~0.5V |
| Reverse recovery time | ~30 µs | ~10 ns |
| Max current | 1A | 5A |
| Power loss at 2A | 2.2W | 1.0W |

- Lower Vf → more voltage available to the buck converters
- Fast recovery → essential for the freewheeling diode role at 150 kHz
- Higher current rating → same diode used in both bridge and freewheeling positions, simplifying procurement

---

## Inductor: 33 µH / 5A (vs. calculated 37 µH)

The calculated minimum inductance was 37 µH. Standard inductor values near this are 33 µH and 47 µH.

**Why 33 µH was chosen over 47 µH:**
- 33 µH → slightly higher ripple current (0.66A vs 0.53A) — both are in continuous conduction mode
- 33 µH is more commonly stocked; 47 µH in 5A rating is harder to find
- At 5A current rating, either value handles peak load without saturation

**Critical requirement:** Current rating must be ≥ 5A. A 33 µH / 1A inductor (common in small modules) will saturate immediately at 2A load and fail.

---

## Filter Capacitor: 10,000 µF / 35V

**Why 10,000 µF:** Calculated minimum for 2V ripple at 2A from 50Hz supply. Smaller values increase ripple, which reduces minimum input voltage to the buck converters.

**Why 35V rating:** The rectified peak voltage is ~16V DC. Component datasheets recommend derating capacitors to 50–70% of rated voltage for longevity. 35V / 16V = 2.2× derating — well within the safe range.

**Alternative considered:** Two 4700 µF capacitors in parallel (9400 µF total) — acceptable if a single 10,000 µF isn't available, but the single large capacitor is preferred for lower ESR.

---

## Transformer: 12V AC, 2A

**Why 12V secondary:**
- After rectification: ~16V DC peak
- Provides adequate headroom for both 5V and 3.3V buck converter inputs
- Step-down ratio keeps rectifier and capacitor voltage ratings manageable

**Why not use a 9V transformer:**
- 9V AC rectified = ~11.7V peak
- After Schottky drop (1V) and ripple trough (2V): minimum ~8.7V
- Still sufficient, but less margin for line voltage variation

**Why not use a 15V transformer:**
- 15V AC rectified = ~20V peak
- Slightly more heat in the LM2596, higher voltage stress on capacitors
- 12V is the sweet spot for this application

**Transformer provides galvanic isolation** — the single most important safety feature for mains-connected automation in outdoor/agricultural environments.

---

## PCB: Dotted PCB (Prototype Stage)

A dotted (perforated) PCB was used for the prototype. For production or next revision, a proper two-layer PCB is recommended with:
- Dedicated ground plane
- Short, wide traces for high-current paths (≥ 2mm for 2A)
- Decoupling capacitors placed physically adjacent to IC pins
- Thermal relief pads for the transformer and LM2596 pins
