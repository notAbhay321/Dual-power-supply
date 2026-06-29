# Future Scope

The current design is a functional, tested prototype. Several improvements are planned for future versions.

---

## Planned Enhancements

### 1. EMI Shielding and Layout Optimization
The LM2596 switching at 150 kHz generates electromagnetic interference that can affect the ESP32's ADC on long signal wires. Future revisions should:
- Move to a proper PCB with a ground plane
- Add a ferrite bead on the 3.3V output rail
- Use shorter, thicker traces for high-current paths
- Add an EMI filter on the AC input (common-mode choke + X/Y capacitors)

### 2. Power Telemetry via ESP32
Add real-time monitoring of the supply itself:
- INA219 current sensor IC on the 5V and 3.3V rails
- Report voltage, current, and power to an MQTT broker or a local web dashboard
- Alert when voltage drops below safe thresholds (brownout prediction)

### 3. Adaptive Power Management
- Implement sleep mode synchronization between the ESP32 and the power supply
- During low-activity periods (no sensor polling, no relay switching), throttle the system to reduce thermal load on filter capacitors
- Extend electrolytic capacitor lifespan by reducing average ripple current

### 4. PCB Miniaturization
- Migrate from dotted PCB to a compact two-layer PCB
- Target form factor: < 80mm × 60mm
- Place decoupling capacitors physically close to LM2596 pins (reduces switching noise)

### 5. Battery Backup
- Add a LiPo/Li-Ion charge management stage (TP4056 or BQ24195)
- Automatic switchover when mains power fails
- Keeps the ESP32 running during power outages (important for water pump control)

### 6. Solar / Off-Grid Support
- Replace transformer stage with a solar charge controller input
- Add MPPT (Maximum Power Point Tracking) for 12V solar panel input
- Enables deployment in farms without grid access

### 7. Enhanced Protection Circuits
- TVS diode on AC input (surge suppression)
- Auto-resetting polyfuse instead of glass fuse
- Reverse polarity protection on DC outputs
- Overcurrent latch with LED fault indicator

---

## Societal Impact

A reliable power supply directly enables the automation of water-critical systems:

| Application | Impact |
|-------------|--------|
| Smart water tank | Prevents overflow (water waste) and pump dry-run (motor damage) |
| Precision irrigation | Accurate soil moisture → correct watering → better crop yield |
| Home automation | Lower maintenance cost; fewer hardware failures |
| Agricultural IoT | Enables off-grid automation with solar extension |

Stable power infrastructure is foundational to IoT-based resource conservation. By ensuring sensors and controllers operate correctly, this design supports reduced water usage, lower electricity bills, and improved reliability in both urban and rural automation contexts.

---

## Conclusion Summary

| Objective | Status |
|-----------|--------|
| Stable 5V @ 2A output | ✅ Achieved |
| Stable 3.3V @ 1A output | ✅ Achieved |
| Ripple < 30 mV on 3.3V rail | ✅ Achieved |
| Handle ESP32 WiFi peak (250–500 mA) | ✅ No resets observed |
| Galvanic isolation from 230V | ✅ Transformer-based |
| 24/7 continuous operation | ✅ Stable in testing |
| Cost under ₹600 | ✅ ~₹583 total BOM |
