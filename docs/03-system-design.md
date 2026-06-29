# System Design

## Architecture Overview

The supply chain converts 230V AC mains into two regulated DC rails through four sequential stages:

```
[230V AC Mains]
      │
      ▼
[Step-Down Transformer]  ← Galvanic isolation + voltage reduction
      │  ~12V AC
      ▼
[Full-Bridge Rectifier]  ← SR560 Schottky diodes
      │  Pulsating DC
      ▼
[Filter Capacitors]      ← 10,000 µF bulk + 470 µF local
      │  ~16V DC (smoothed)
      ├──────────────────────────────────┐
      ▼                                  ▼
[LM2596-ADJ Buck #1]            [LM2596-ADJ Buck #2]
      │  5V @ 2A                         │  3.3V @ 1A
      ▼                                  ▼
[Relay Modules]              [ESP32 Microcontroller]
[Ultrasonic Sensor]          [Soil Moisture Sensor]
[Water Pump Control]
```

---

## Functional Block Diagram (Text)

```
AC Input (230V 50Hz)
  └─► Step-Down Transformer (220V → 12V AC)
        └─► Bridge Rectifier + Filter (AC → ~16V DC)
              ├─► LM2596 Buck Converter #1
              │     └─► 5V Rail
              │           ├─► Relay 1 (Water Pump)   [GPIO]─┐
              │           ├─► Relay 2 (Sprinkler)    [GPIO]─┤─► ESP32
              │           ├─► Relay 3 (Spare)        [GPIO]─┘
              │           └─► Ultrasonic Sensor (HC-SR04)
              │
              └─► LM2596 Buck Converter #2
                    └─► 3.3V Rail
                          ├─► ESP32 (VCC)
                          └─► Soil Moisture Sensor
```

---

## Design Alternatives Considered

A morphological analysis was done across four functional stages:

### Stage 1 — Step-Down / Isolation

| Option | Pros | Cons | Selected? |
|--------|------|------|-----------|
| Step-down transformer | Safe galvanic isolation, robust | Bulky, heavier | ✅ Yes |
| Capacitive dropper | No isolation, cheaper | Dangerous for outdoor use | ❌ No |
| SMPS primary-side | Compact, efficient | Complex design, high EMI risk | ❌ No |

### Stage 2 — Rectification

| Option | Vf Drop | Speed | Selected? |
|--------|---------|-------|-----------|
| 1N4007 silicon bridge | ~1.1V | Slow | ❌ No |
| SR560 Schottky bridge | ~0.5V | Fast | ✅ Yes |

SR560 selected for lower conduction losses and fast switching compatibility with SMPS stages.

### Stage 3 — Filtering

| Option | Capacitance | Notes | Selected? |
|--------|-------------|-------|-----------|
| 1000–2200 µF | Lower | More ripple at 2A load | ❌ No |
| 10,000 µF | Higher | Calculated minimum for 2V ripple at 2A | ✅ Yes |

### Stage 4 — Voltage Regulation

| IC | Type | Efficiency | Max Current | Heat | Selected? |
|----|------|------------|-------------|------|-----------|
| IC 7805 | Linear | ~45% | 1A | Very High | ❌ No |
| AMS1117-3.3 | LDO | ~60–70% | 1A | Medium | ❌ No |
| MP1584 | Buck | ~92% | 3A | Very Low | ⚠️ Alternative |
| **LM2596-ADJ** | **Buck** | **88–90%** | **3A** | **Low** | ✅ **Yes** |

**Why LM2596 over MP1584:** LM2596 has a larger package, easier to hand-solder on dotted PCB, proven datasheet application circuit, and identical current rating at slightly lower cost.

---

## Operational Flowchart

```
START
  │
  ▼
230V AC Input Applied
  │
  ▼
Step-Down Transformer
(Isolation + Voltage Reduction to ~12V AC)
  │
  ▼
Bridge Rectifier (SR560)
(AC → Pulsating DC)
  │
  ▼
Filter Capacitors
(10,000 µF + 470 µF → Smooth DC ~16V)
  │
  ▼
LM2596 Buck Converter Stages
(Generate 5V and 3.3V)
  │
  ▼
Output Voltage Stable?
  ├── NO  → Fault (check components / overload)
  └── YES
        │
        ▼
Power Supplied to ESP32, Sensors & Relays
  │
  ▼
Continuous Operation (24/7)
  │
  ▼
END (Power removed)
```

---

## Safety Design Decisions

1. **Transformer isolation** — complete galvanic break between 230V mains and DC output; essential for outdoor/agricultural use
2. **Fuse on AC input** — protects against primary-side faults
3. **35V-rated capacitors** at 16V DC bus — 2× voltage headroom margin
4. **LM2596 internal thermal shutdown** — prevents damage on overload
5. **SR560 Schottky as freewheeling diode** — handles fast switching without thermal stress
