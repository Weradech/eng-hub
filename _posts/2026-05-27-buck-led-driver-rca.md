---
title: "Buck LED Driver Field Failure RCA — A Systematic Approach"
date: 2026-05-27 10:00:00 +0700
categories: [Circuit, RCA]
tags: [led, circuit, rca, buck, power-electronics, field-failure]
---

> **TL;DR** — Always gather 4 symptoms before proposing a hypothesis. Real case: GSC5_HL freewheel diode 1SS355 undersized for peak current → replaced with B240A, resolved.

---

![A buck LED-driver field failure RCA: one symptom forks into four candidate causes, each isolated by datasheet and worst-case math, converging on an undersized freewheel diode and the B240A fix]({{ "/assets/img/2026-05-27/buck-led-driver-rca.svg" | relative_url }})
_One symptom, four candidate causes — datasheet math, not guesswork, finds the undersized freewheel diode._

## Why Field Failure RCA Is Hard

Field failures differ from lab failures because:
- No scope waveforms to examine
- Parts have been exposed to multiple stressors simultaneously (temperature + vibration + voltage transients)
- Customers describe symptoms in general terms ("won't turn on", "flickering", "gets hot")

**Golden rule: Never propose a root cause from fewer than 4 symptoms.**

---

## 4 Symptoms Checklist

Answer these before starting any RCA:

| # | Question | Example Answer |
|---|----------|---------------|
| 1 | What is the failure mode? | LED permanently off / flickering / reduced brightness |
| 2 | When did it occur? (runtime hours) | After 200h / cold start / thermal cycle |
| 3 | Environmental conditions? | Temperature, humidity, vibration |
| 4 | Pattern — random or systematic? | All units from batch X / random |

---

## Buck LED Driver Anatomy

```
VIN ──[L]──┬── VOUT ── LED String
           │
          [D]  ← Freewheel Diode (most common failure point)
           │
          GND
```

**Most frequently failed components (from field experience):**

1. **Freewheel Diode** — insufficient peak current rating
2. **Inductor** — core saturation at high current
3. **Sense Resistor** — drift causing incorrect current regulation
4. **Control IC** — latch-up from transients

---

## Case Study — GSC5_HL Freewheel Diode

### Gathered Symptoms

| Symptom | Detail |
|---------|--------|
| Failure mode | LED permanently off on select units |
| Runtime | 50–200h |
| Environment | Automotive interior, Tamb 85°C max |
| Pattern | Systematic — ~15% failure rate within same batch |

### Hypothesis

Systematic + thermal → suspect **thermal stress on an undersized component**

**Verification sequence:**
1. Look up actual datasheet (do not rely on memory)
2. Calculate worst-case current
3. Compare against rated values

### Root Cause

**1SS355 (original):**
- IF(avg) = 100 mA
- IFSM (surge) = 1 A

**Actual circuit demand:**
- Peak inductor current = 850 mA (L = 47 µH, f = 200 kHz)
- Estimated junction temp > 125°C at Tamb = 85°C

→ **1SS355 cannot sustain surge current under hot conditions**

### Fix — B240A

| Spec | 1SS355 | B240A |
|------|--------|-------|
| Type | Small signal | Schottky power |
| IF(avg) | 100 mA | 2 A |
| IFSM | 1 A | 60 A |
| VF | 0.9 V | 0.4 V |
| Package | SOD-323 | SMA |

Lower VF also means improved efficiency and reduced heat dissipation.

### Pre-Implementation Verification

```
1. Check B240A datasheet at TJ = 125°C — is IF still sufficient?
2. Calculate power dissipation: Pd = VF × IF(avg)
3. Verify thermal resistance: θJA
4. Confirm footprint compatibility (SMA vs SOD-323 → layout change required)
```

---

## RCA Methodology Summary

```
Receive complaint
    ↓
Gather 4 symptoms (never skip)
    ↓
Identify top 3 suspect components
    ↓
Verify datasheet values (not from memory)
    ↓
Calculate worst-case → compare against rating
    ↓
Propose fix + verify math is self-consistent
    ↓
Change ONE component at a time
    ↓
EVT → Verify → Close
```

---

## Common Anti-patterns

| ❌ Avoid | ✅ Instead |
|---------|-----------|
| Propose root cause from 1 symptom | Always gather 4 |
| Use typical datasheet values | Use worst-case |
| Change multiple components at once | One change at a time |
| Ignore footprint compatibility | Check before specifying fix |

> This methodology applies to any DC-DC converter topology, not just LED drivers.
