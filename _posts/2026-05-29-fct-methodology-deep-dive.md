---
title: "FCT Methodology Deep Dive — Functional Test from Power-On to Calibration"
date: 2026-05-29 14:30:00 +0700
categories: [NPI, Test]
tags: [fct, functional-test, test-engineering, pcba, firmware, calibration, burn-in, ni-teststand, labview]
---

> **TL;DR** — FCT is the *final gate before Box Build*. It verifies the board actually performs its intended function — power sequencing, communication, sensor read, actuator drive, firmware boot, calibration. NRE ฿58K–203K. Per-unit cost ฿100/test point. Combined with ICT, achieves 98%+ defect coverage.

![FCT is the final gate before Box Build: a board runs through power-on, communication, sensor read, actuator drive, firmware boot, and calibration; passing all six releases it. ICT checks parts are present, FCT checks the board functions, and together they reach 98 percent-plus defect coverage]({{ "/assets/img/2026-05-29/fct-methodology-deep-dive.svg" | relative_url }})
_ICT asks “is it built right?”; FCT asks “does it work?” — a board needs both before it ships._

This is a companion to the [ICT Methodology Deep Dive]({% post_url 2026-05-29-ict-methodology-deep-dive %}). ICT verifies components are present and correct; FCT verifies the board *works*.

---

## What FCT Actually Tests

FCT (Functional Test) is a **system-level test** that simulates the end product's real use case: apply power and input signals, measure outputs, decide pass/fail against specification.

| Aspect | ICT | FCT |
|--------|-----|-----|
| Level | Component | System |
| Verifies | Parts present + correct | Board *works* |
| Power | Limited rails only | Full operating power |
| Firmware | Not applicable | Required (flashed first) |
| Time | 5–30 sec | 10 sec – 15 min |

Position in NPI flow:

```
SMT → AOI → ICT → [Firmware Flash] → FCT → Calibration → [Burn-in] → Box Build → Final QC
                       ↑                 ↑
                  Often part of FCT  Sometimes separate station
```

**FCT is the final electrical gate before mechanical assembly.** Any board passing FCT should be ready to ship as-is.

---

## Test Categories — What FCT Covers

### 1. Power-On Sequence

- **Inrush current** — peak startup current within spec
- **Rail voltage** — V_cc / V_dd / 3.3V / 5V / 12V within tolerance, ripple acceptable
- **Power sequencing** — multi-rail systems must power up in correct order (critical for FPGA/SoC)
- **Quiescent current** — idle current matches datasheet
- **Brown-out / under-voltage response** — correct behaviour during supply dip

### 2. Go / No-Go (Binary Pass/Fail)

Simple yes/no checks for basic products:

- LED on/off
- Buzzer sound/silent
- Relay click/no-click
- Display shows/doesn't show
- Best for: LED bars, relay modules, sensor breakouts

### 3. Parametric Test (Tolerance-Based)

| Parameter | Example Spec |
|-----------|--------------|
| Voltage output | LED driver 12.0 V ± 0.2 V |
| Current consumption | Idle vs active mode per spec |
| Frequency | Oscillator/PWM ± 2% |
| Timing | Pulse width, rise/fall time |
| Gain | Amplifier 10–100 kHz response |

Best for analog signal chains, power supplies, sensor conditioning circuits.

### 4. Functional Test (Use Case Simulation)

- **Communication** — send command via UART/SPI/I²C/CAN/USB → expect response
- **Sensor read** — apply simulated stimulus (temperature simulator, pressure pump) → check sensor output
- **Actuator drive** — board commands motor/relay/solenoid → external sensor confirms action
- **Display sequence** — board outputs pattern → camera/photodiode validates

### 5. Firmware-Related Tests

- **Boot test** — bootloader + firmware boot within spec time
- **Memory test** — RAM/Flash read/write integrity
- **Watchdog test** — WDT reset behaviour verified
- **OTA update test** — wireless firmware update flow

### 6. Communication Interface Verification

| Interface | What FCT Verifies |
|-----------|-------------------|
| UART | Baud rate accuracy, framing, echo loopback |
| SPI | Master/slave clock + data integrity |
| I²C | Address ACK, read/write, pull-up correct |
| CAN | Bit timing, arbitration, error frames |
| USB | Enumeration, descriptor, bulk transfer |
| Ethernet | Link up, MAC echo, PHY register access |
| BLE / Wi-Fi | Advertise/scan, RSSI, throughput |
| LIN | Master schedule, slave response timing |

### 7. RF / Wireless (Specialized)

- TX power output (dBm)
- RX sensitivity
- Frequency error
- EVM (Error Vector Magnitude)
- Spurious emission

### 8. End-of-Line (EOL) Combined Test

Single station performs power + function + calibration + serial number burn in one cycle. Common in high-volume production.

---

## Fixture Types

### A. Bed-of-Nails + Functional Pogo

Probes contact dedicated FCT pads. Power injection + signal probing happen simultaneously.

| Parameter | Value |
|-----------|-------|
| NRE | ฿40K – ฿200K |
| Cycle time | 10–60 sec |
| Best for | Production volume |

### B. Edge Connector / IO Connector Mating

Uses ZIF socket or pin header mating the actual product connector. Simulates field connection.

| Parameter | Value |
|-----------|-------|
| NRE | ฿15K – ฿80K |
| Cycle time | 30 sec – 5 min (manual load) |
| Best for | LVHM, prototype, NPI |

Example: OBD-II connector for automotive controller test.

### C. Custom Test Rig (Full System Mock)

Simulates complete end-product environment — load, sensor input, communication peer.

| Parameter | Value |
|-----------|-------|
| NRE | ฿80K – ฿500K+ |
| Cycle time | 1–10 min |
| Best for | Complex multi-domain products |

Examples:
- **LED bar FCT** — constant current load + lux meter + temperature chamber
- **Controller FCT** — CAN bus simulator + relay load bank + serial terminal

### D. Multi-DUT Cluster Fixture

Tests 4/8/16 boards in parallel. Increases throughput, NRE scales nearly linearly with channel count.

- Best for very high volume (>50K/month)

---

## Equipment Stack

### Hardware

| Equipment | Use |
|-----------|-----|
| Programmable PSU (Keysight E36300, Rigol DP800) | V_cc supply + measure |
| Electronic load (Keysight EL34143A) | Constant current/voltage/power load |
| DMM (Keysight 34465A) | Precision V/I measurement |
| Oscilloscope (Picoscope USB) | Timing + waveform |
| Function generator | Input stimulus |
| Logic analyzer (Saleae) | Digital protocol decode |
| CAN/LIN simulator (Vector, PEAK) | Bus message injection |
| RF analyzer (Tektronix, Keysight) | Wireless test |
| Temperature chamber | Environmental stress |

### Software Frameworks

| Tool | Strength |
|------|----------|
| **NI TestStand + LabVIEW** | Industry standard, sequencer + UI + report |
| **NI VeriStand** | Real-time HIL test |
| **Python + pytest + pyvisa** | Open source, flexible, common in LVHM |
| **MATLAB + Test Manager** | Math-heavy parametric |
| **Custom C# / WPF** | Legacy automotive |
| **Go / Rust CLI tools** | Embedded / edge test station |

> Default recommendation for flexible LVHM: **Python + pyvisa + custom GUI** — no NI license, full version control, reuses engineering team's existing skills.

---

## Firmware Flash During FCT

Most FCT stations include firmware flash in the same cycle:

1. Board placed in fixture
2. Power on → verify rail voltage
3. **Flash firmware** via SWD/JTAG/ISP/UART bootloader
4. Reset → boot test
5. Run functional sequence
6. **Burn unique data** — serial number, MAC address, calibration coefficients
7. Verify burn-in successful (read back)
8. Final pass → label print + barcode scan

Common programmers: ST-Link, J-Link, OpenOCD, custom ISP programmer

---

## Calibration Step

Many analog products require per-unit calibration during FCT:

| Product | Calibration |
|---------|-------------|
| LED driver | Current setpoint trim to ±1% |
| Sensor module | Zero/span calibration → coefficient stored in EEPROM |
| Audio amplifier | Gain trim |
| Power meter | Voltage/current shunt calibration |

Calibration coefficients are written to Flash/EEPROM per unit, then tied to the serial number/barcode. Lost cal data = unusable product.

---

## Extended FCT — Burn-in, HALT, HASS

| Test | Duration | Purpose |
|------|----------|---------|
| **Burn-in** | 24–168 hr at elevated temp | Screen infant mortality |
| **HALT** (Highly Accelerated Life Test) | Days | Find design weakness during NPI |
| **HASS** (Highly Accelerated Stress Screen) | Hours | Production weed-out for high-reliability products |

HALT/HASS typically run in lab during NPI phase. Burn-in may live on the production line for safety-critical products.

---

## NRE Breakdown

| Item | Cost (THB) | Notes |
|------|-----------|-------|
| Fixture mechanical (pogo + frame or connector mating) | 15,000 – 80,000 | Depends on complexity |
| Test program development (Python/LabVIEW) | 20,000 – 60,000 | 10–20 engineer-days |
| Firmware flash integration | 5,000 – 15,000 | SWD/JTAG setup + burn routines |
| Calibration routine (if required) | 10,000 – 30,000 | Per parameter |
| Debug + golden sample validation | 5,000 – 10,000 | 2–3 days |
| Documentation (FCT plan + SOP + report format) | 3,000 – 8,000 | |
| **Typical total** | **฿58,000 – ฿203,000** | |

### Per-Unit FCT Cost

- **Base rate: ฿100 per test point per unit**
- Add ฿20–50/unit for calibration trim/burn
- Add ฿10–20/unit for firmware flash + serial burn

---

## Time per Board

| Product Type | FCT Time |
|--------------|----------|
| Simple LED / relay | 10–30 sec |
| Sensor module | 30–60 sec |
| MCU + sensor + comm | 1–3 min |
| Multi-rail power supply | 2–5 min |
| Complex controller (CAN/multi-IO) | 3–10 min |
| Wireless (BLE/Wi-Fi cert) | 5–15 min |

Throughput planning: one station produces 60–360 boards/hr (simple) or 12–60 boards/hr (complex).

---

## Decision Framework — FCT Scope

```
Product complexity?
├─ Single function (LED on/off, relay click) → Go/No-Go FCT (cheap)
├─ Multi-rail power + IO → Parametric + interface test
├─ MCU + firmware → Flash + boot + comm test
├─ Analog precision → Parametric + Calibration
└─ Wireless / safety-critical → Full param + cert + burn-in

Volume?
├─ < 100 pcs (proto) → Manual bench test (no fixture)
├─ 100–1,000 pcs → Simple ZIF/connector fixture
├─ 1,000–10,000 pcs → BoN FCT + automation
└─ > 10,000 pcs → Multi-DUT cluster + in-line

ICT done?
├─ Yes → FCT focuses on function (components already verified)
└─ No → FCT must include basic power-on safety + missing-part screen
```

---

## When to Use FCT-Only (Skip ICT)

- Volume < 200 pcs/year
- Simple Go/No-Go product
- Customer accepts coverage gap
- Prototype / NPI stage

## When Both ICT and FCT are Mandatory

- Safety-critical (medical, automotive ISO 26262)
- High-mix LED bar / driver products
- Customer audit requires component-level traceability
- Volume > 1,000 pcs (BoN ICT NRE pays back)

---

## Coverage Gap — ICT vs FCT

| Defect Type | ICT | FCT |
|-------------|-----|-----|
| Wrong component value | ✅ | ⚠️ (only if affects function) |
| Solder bridge | ✅ | ⚠️ (only if affects function) |
| Firmware bug | ❌ | ✅ |
| Timing / clock | ❌ | ✅ |
| Communication protocol fail | ❌ | ✅ |
| Calibration drift | ❌ | ✅ |
| EMI / noise | ❌ | ⚠️ (needs RF setup) |
| Thermal performance | ❌ | ⚠️ (needs chamber) |
| Marginal in-spec performance | ❌ | ✅ |
| Wrong firmware version | ❌ | ✅ |
| Sensor calibration error | ❌ | ✅ |

---

## FCT Failure Analysis Workflow

When FCT fails:

1. **Log fault code** — which test step failed, measured value vs spec range
2. **Visual inspection** — focused re-AOI of failing area
3. **Re-run FCT** — intermittent or repeatable?
4. **Probe failing rail/signal** with manual oscilloscope
5. **Compare to golden sample** — keep golden waveform/trace for reference
6. **Trace back logic** — FCT fail but ICT pass → likely firmware, timing, or parametric
7. **FA report** to customer + corrective action

---

## Anti-Patterns

| ❌ Avoid | ✅ Instead |
|---------|-----------|
| Quote FCT as flat per-board fee | Itemize NRE + per-unit + per-TP |
| Skip golden sample baseline | Capture and version-control golden waveforms |
| Hard-code spec limits in test code | Parameterize from product config file |
| One station per product variant | Modular fixture + parameterized program |
| Send firmware flash to separate station | Integrate in FCT cycle for traceability |
| Ignore calibration drift over time | Verify cal coefficients monthly on golden DUT |

---

## Related Posts

- [ICT Methodology Deep Dive]({% post_url 2026-05-29-ict-methodology-deep-dive %}) — component-level test counterpart
- [ICT Assessment Pattern]({% post_url 2026-05-27-ict-assessment-pattern %}) — NPI-stage ICT evaluation
- [PCBA RFQ Costing Pattern]({% post_url 2026-05-27-pcba-rfq-costing-pattern %}) — how to price FCT in quotation
