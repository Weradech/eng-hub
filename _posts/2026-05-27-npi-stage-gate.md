---
title: "NPI Stage Gate — PoC → EVT → DVT → PVT → MP ทำไมต้องมีทุก Gate?"
date: 2026-05-27 13:00:00 +0700
categories: [NPI, Process]
tags: [npi, stage-gate, evt, dvt, pvt, product-development]
---

> **TL;DR** — Stage Gate ไม่ใช่ bureaucracy แต่คือ risk firewall แต่ละ gate ปิด risk ประเภทหนึ่งก่อนลงทุนเพิ่ม

---

## ทำไมต้องมี Stage Gate?

ต้นทุนแก้ปัญหาเพิ่มแบบ exponential ตามระยะของ project:

```
Design stage:    ฿1,000   (แก้ schematic)
EVT stage:       ฿10,000  (แก้ layout + respin)
DVT stage:       ฿100,000 (แก้ tooling/mold)
Production:      ฿1,000,000+ (recall, rework, warranty)
```

**Gate = จุดตัดสินใจว่าพร้อมลงทุนขั้นต่อไปหรือยัง**

---

## 5 Stages

### PoC — Proof of Concept
**เป้าหมาย:** พิสูจน์ว่า concept ทำงานได้จริงในหลักการ

**Deliverables:**
- Breadboard / dev kit prototype
- Key function demo (ไม่ต้องครบทุก feature)
- BOM draft (ไม่ต้อง costed)
- Go/No-Go decision

**Gate ออก:** Core function works → ลงทุน layout จริง

---

### EVT — Engineering Validation Test
**เป้าหมาย:** validate ว่า design ทำงานได้ตาม spec ทุกข้อ

**Deliverables:**
- First PCB spin (hand-built หรือ proto house)
- Schematic + Layout complete
- BOM with approved vendors
- Test report vs. spec
- Failure analysis ทุกจุดที่ fail

**Gate ออก:** All critical specs pass → ลงทุน DVT tooling

**EVT fail patterns:**
- Power integrity (noise, ripple)
- Thermal ไม่ผ่าน (component เกิน Tjmax)
- EMI fail pre-scan
- Firmware stability

---

### DVT — Design Validation Test
**เป้าหมาย:** validate ว่า design + process ผลิตได้จริง และผ่าน reliability

**Deliverables:**
- Production-intent PCB (จาก production vendor)
- Tooling / fixture / jig complete
- Reliability test: temp cycle, humidity, vibration
- Regulatory compliance: CE/FCC/TIS pre-test
- DFM/DFA sign-off
- SOP draft

**Gate ออก:** Reliability pass + yield ≥ target → ลงทุน PVT

**DVT คือ stage ที่แพงที่สุดถ้า fail** เพราะมี tooling cost แล้ว

---

### PVT — Production Validation Test
**เป้าหมาย:** validate ว่า production line ผลิตได้ที่ volume จริง พร้อม ship

**Deliverables:**
- Pilot run 50–500 pcs (ขึ้นกับ volume)
- Yield data จาก production line
- Test coverage verify (ICT/FCT pass rate)
- Packaging + label verify
- Golden sample + limit sample เซ็ต

**Gate ออก:** Yield ≥ target, 0 critical defect → MP ramp

---

### MP — Mass Production
**เป้าหมาย:** ผลิตซ้ำได้อย่างสม่ำเสมอ ภายใต้ cost target

**Ongoing:**
- SPC monitoring (Cpk ≥ 1.33)
- ECR process สำหรับ continuous improvement
- End-of-life planning

---

## Gate Criteria Template

| Gate | Must Pass | Should Pass | Monitor |
|------|-----------|-------------|---------|
| PoC → EVT | Core function | Performance ballpark | BOM cost |
| EVT → DVT | All specs | Yield > 70% | Regulatory |
| DVT → PVT | Reliability | Yield > 90% | Cost target |
| PVT → MP | Yield ≥ target | 0 critical defects | Supplier readiness |

---

## Common Mistakes

| Mistake | Impact |
|---------|--------|
| ข้าม EVT ไป DVT เลย (รีบ) | DVT fail → cost สูง + delay มากกว่าเดิม |
| DVT ด้วย prototype board แทน production board | Yield ใน PVT ต่างจาก DVT มาก |
| ไม่ทำ DFM review ก่อน DVT | Tooling ต้อง rework = ค่าใช้จ่ายสูง |
| Pass gate แบบ "majority pass" | Minority fail = field failure ในอนาคต |

---

## NPI Timeline Reference

```
PoC:  2–4 สัปดาห์
EVT:  4–8 สัปดาห์ (รวม PCB lead time 2–3 สัปดาห์)
DVT:  8–12 สัปดาห์ (รวม reliability test)
PVT:  4–6 สัปดาห์
MP:   ongoing
─────────────────────
Total NPI: 18–30 สัปดาห์ (4.5–7.5 เดือน) สำหรับ product ซับซ้อนปานกลาง
```
