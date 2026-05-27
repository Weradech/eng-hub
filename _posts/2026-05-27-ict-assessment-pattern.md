---
title: "ICT Assessment Pattern — ประเมิน In-Circuit Test ตั้งแต่ขั้น NPI"
date: 2026-05-27 12:00:00 +0700
categories: [NPI, Test]
tags: [ict, dft, test-engineering, npi, pcba, nre]
---

> **TL;DR** — 5 ขั้นตอน: รับไฟล์ → นับ TP/Via → ประเมิน DFT 9 ข้อ → คำนวณ NRE → สร้าง SOP report พร้อม propose ให้ลูกค้า

---

## ทำไม ICT ต้องประเมินตั้งแต่ NPI Stage?

ICT fixture สร้างครั้งเดียวใช้ตลอด production ถ้าออกแบบผิดหรือ layout ไม่รองรับ DFT:
- Fixture ทำใหม่ = NRE เพิ่ม ฿50,000–500,000
- ต้อง ECR แก้ layout = delay 2–4 สัปดาห์
- Coverage ต่ำ = escape defect ถึงลูกค้า

**Golden rule: ประเมิน ICT ตอนที่ยังแก้ layout ได้ฟรี**

---

## 5-Step ICT Assessment Process

### Step 1 — รับและ Review ไฟล์

ไฟล์ที่ต้องได้จากลูกค้า/design team:

| ไฟล์ | Format | ใช้ทำอะไร |
|------|--------|-----------|
| Gerber / ODB++ | .gbr / .tgz | นับ TP, via, pad |
| BOM | Excel / CSV | map component → testability |
| Schematic | PDF / Altium | เข้าใจ net topology |
| Assembly drawing | PDF | ดู component side, keep-out |

---

### Step 2 — นับ Test Point (TP) และ Via

**นับจาก Gerber layer:**

```
TP ด้าน Top = นับจาก Top Copper (ที่มี pad drill)
TP ด้าน Bottom = นับจาก Bottom Copper
Via accessible = Via ที่ไม่ถูก component บัง (top view)
```

**Coverage estimate:**
```
Node count = จำนวน net ที่ต้องวัด
TP count = จำนวน physical test point
Coverage (%) = (TP count / Node count) × 100
```

Target: **≥ 85% node coverage** สำหรับ production board

---

### Step 3 — DFT Checklist 9 ข้อ

| # | DFT Criteria | Pass Condition |
|---|-------------|----------------|
| 1 | Test point size | ≥ 1.0mm diameter (Ø) |
| 2 | Test point pitch | ≥ 2.54mm center-to-center |
| 3 | TP ห่างจาก component | ≥ 1.5mm clearance |
| 4 | TP ห่างจากขอบ board | ≥ 3.0mm |
| 5 | TP บน bottom side | ≥ 60% ของ TP ทั้งหมด (single-side fixture) |
| 6 | Via ที่ใช้เป็น TP | ห้าม tented via |
| 7 | Power / Ground access | ต้องมี TP บน main rails ทุกตัว |
| 8 | Crystal / Oscillator | ต้องมี disable point หรือ bypass |
| 9 | Programming header | ต้องมี JTAG/SWD/ISP pad accessible |

**Scoring:**
- 9/9 Pass → ICT เต็มรูปแบบได้
- 6–8 Pass → ICT ได้แต่ coverage จำกัด
- < 6 Pass → แนะนำ FVT หรือ AOI แทน + ECR

---

### Step 4 — คำนวณ NRE

```
Fixture NRE = Base cost + (TP count × rate) + Engineering hours

ตัวอย่าง:
Base cost      = ฿25,000
TP count = 120 × ฿200/point = ฿24,000
Engineering    = 16h × ฿800/h = ฿12,800
───────────────────────────────────────
Total NRE      = ฿61,800
```

**Lead time:** 4–6 สัปดาห์หลัง approve Gerber final

---

### Step 5 — สร้าง Assessment Report

Report ประกอบด้วย:
1. **Board overview** — ขนาด, layer count, component count
2. **TP Summary table** — top/bottom count, coverage %
3. **DFT Score** — 9 ข้อ pass/fail พร้อม recommendation
4. **NRE Estimate** — itemized
5. **Recommendation** — ICT / FVT / AOI / Combined strategy

---

## Case Study — MIRA R3

| Parameter | Value |
|-----------|-------|
| Board size | 120 × 80mm, 4-layer |
| Component count | 187 |
| TP count (bottom) | 94 |
| Via accessible | 38 |
| Node count | 118 |
| Coverage | 112/118 = **94.9%** ✅ |
| DFT Score | 8/9 (Crystal disable point missing) |
| Recommendation | ICT ได้เลย + เพิ่ม crystal bypass resistor DNI |
| NRE Estimate | ฿68,500 |

---

## Anti-patterns

| ❌ อย่าทำ | ✅ ทำแทน |
|-----------|---------|
| ประเมินจาก BOM อย่างเดียว | ดู Gerber จริงเสมอ |
| นับแค่ TP labeled "TP" | รวม via ที่ accessible ด้วย |
| Quote NRE แบบ lump sum ไม่แจก | Itemize ให้ลูกค้าเห็น |
| ไม่เช็ค tented via | Via ทุก tented = ใช้เป็น TP ไม่ได้ |
