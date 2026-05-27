---
title: "Buck LED Driver Field Failure RCA — วิธีวิเคราะห์ความล้มเหลวในสนาม"
date: 2026-05-27 10:00:00 +0700
categories: [Circuit, RCA]
tags: [led, circuit, rca, buck, power-electronics, field-failure]
---

> **TL;DR** — ก่อนเสนอ hypothesis ต้อง gather 4 symptoms ก่อนเสมอ จาก Case จริง GSC5_HL: freewheel diode 1SS355 ทนกระแสไม่พอ → เปลี่ยนเป็น B240A แก้ได้

---

## ทำไม Field Failure RCA ถึงยาก?

Field failure ต่างจาก lab failure ตรงที่:
- ไม่มี scope waveform ให้ดู
- ชิ้นงานมักถูก stress หลายอย่างพร้อมกัน (temp + vibration + voltage transient)
- ลูกค้าอธิบาย symptom ด้วยคำพูดทั่วไป ("ไฟไม่ติด" "กะพริบ" "ร้อน")

กฎเหล็ก: **อย่าเสนอ root cause ก่อนมี 4 symptoms**

---

## 4 Symptoms Checklist

ก่อน RCA ทุกครั้ง ต้องตอบให้ได้ก่อน:

| # | คำถาม | ตัวอย่างคำตอบ |
|---|-------|---------------|
| 1 | Failure mode คืออะไร? | LED ดับถาวร / กะพริบ / สว่างลดลง |
| 2 | เกิดตอนไหน? (Runtime hours) | หลังใช้งาน 200h / cold start / thermal cycle |
| 3 | Environmental condition? | อุณหภูมิ, ความชื้น, vibration |
| 4 | Pattern — random หรือ systematic? | Unit ที่ X batch เดียวกันทุกตัว / สุ่ม |

---

## Anatomy of a Buck LED Driver

```
VIN ──[L]──┬── VOUT ── LED String
           │
          [D]  ← Freewheel Diode (จุดที่ fail บ่อย)
           │
          GND
```

**Component ที่ fail บ่อยที่สุด (จากประสบการณ์):**

1. **Freewheel Diode** — ทนกระแส peak ไม่พอ
2. **Inductor** — core saturation ที่ current สูง
3. **Sense Resistor** — drift ทำให้ current ผิด
4. **IC Control** — latch-up จาก transient

---

## Case Study — GSC5_HL Freewheel Diode

### Symptoms ที่ได้รับ

| Symptom | Detail |
|---------|--------|
| Failure mode | LED ดับถาวรใน unit บางตัว |
| Runtime | หลังใช้งาน 50–200h |
| Environment | Automotive interior, Tamb 85°C max |
| Pattern | Systematic — batch เดียวกัน ~15% failure rate |

### Hypothesis Generation

จาก symptom = systematic + thermal → สงสัย **thermal stress บน component ที่ spec ต่ำ**

**เช็คลำดับ:**
1. Verify datasheet ของ diode จริง (ไม่ใช่จากความจำ)
2. คำนวณ worst-case current
3. เทียบกับ rating

### Root Cause

**1SS355 (original):**
- IF(avg) = 100mA
- IFSM (surge) = 1A

**Actual circuit demand:**
- Peak inductor current = 850mA (จาก L = 47µH, freq = 200kHz)
- Junction temp estimate = 125°C+ ที่ Tamb 85°C

→ **1SS355 ทน surge ไม่พอ** ใน hot condition

### Fix — B240A

| Spec | 1SS355 | B240A |
|------|--------|-------|
| Type | Small signal | Schottky power |
| IF(avg) | 100mA | 2A |
| IFSM | 1A | 60A |
| VF | 0.9V | 0.4V |
| Package | SOD-323 | SMA |

→ VF ต่ำกว่าด้วย = efficiency ดีขึ้น + ความร้อนลด

### Verify ก่อน Implement

```
1. ดู datasheet B240A ที่ TJ=125°C → IF ยังพอไหม?
2. คำนวณ power dissipation: Pd = VF × IF(avg)
3. เทียบกับ thermal resistance: θJA
4. ยืนยันว่า footprint compatible (SMA vs SOD-323 = ต้องแก้ layout)
```

---

## RCA Methodology Summary

```
รับ complaint
    ↓
Gather 4 symptoms (อย่าข้ามขั้นนี้)
    ↓
ระบุ suspect component (top 3)
    ↓
Verify datasheet จริง (ไม่ใช่จากความจำ)
    ↓
คำนวณ worst-case → เทียบ rating
    ↓
Propose fix + verify math self-consistent
    ↓
Single component fix ก่อน (อย่า change หลาย component พร้อมกัน)
    ↓
EVT → Verify → Close
```

---

## Anti-patterns ที่พบบ่อย

| Anti-pattern | ปัญหา |
|-------------|-------|
| เสนอ root cause จาก 1 symptom | miss true cause, แก้ผิดจุด |
| ใช้ datasheet typical แทน worst-case | pass ที่ lab fail ใน field |
| Change หลาย component พร้อมกัน | ไม่รู้ว่าอะไรแก้ได้จริง |
| ไม่ verify footprint compatibility | layout revision เพิ่ม cost |

---

> Pattern นี้ใช้ได้กับ DC-DC converter ทุกประเภท ไม่ใช่แค่ LED driver ค่ะ
