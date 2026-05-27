---
title: "Paperless GR→IQC Flow — ออกแบบ Trust Signal แทน Stamp"
date: 2026-05-27 17:00:00 +0700
categories: [System, Process]
tags: [wms, iqc, gr, paperless, quality, warehouse]
---

> **TL;DR** — Paperless ไม่ได้แปลว่าลบขั้นตอน แต่แปลว่าเปลี่ยน physical stamp/signature เป็น digital trust signal ที่ traceable และ audit-ready ได้ดีกว่า

---

## ทำไม Paperless?

**ปัญหาของ paper-based GR/IQC:**
- กระดาษหาย → ไม่รู้ว่า lot นี้ผ่าน IQC หรือยัง
- Handwriting อ่านไม่ออก → data quality ต่ำ
- ต้องเดินกระดาษ Store → QA → Accounting → delay 1–2 วัน
- ไม่มี real-time visibility → Manager ไม่รู้สถานะ

**ผลที่ต้องการ:**
- ของเข้า → IQC ตรวจ → เข้าคลัง ทั้งหมดใน system, real-time
- ทีม accounting เห็น GR document ทันทีที่ store รับของ
- QA เปิด phone → เห็น pending inspection queue

---

## 3-Phase Rollout

### Phase 1 — Hybrid (เดือน 1–2)
**เป้าหมาย:** ระบบดิจิทัลทำงานคู่กับกระดาษ เพื่อ build trust

- Store บันทึก GR ทั้งในระบบ **และ** กระดาษ
- QA ตรวจของและบันทึกใน WMS + กรอกกระดาษด้วย
- Cross-check ว่า digital = paper ทุกวัน

**สิ่งที่ต้องเตรียม:**
- QR code ติดที่ receiving dock → สแกนเปิด GR form ทันที
- Tablet/phone ให้ QA ใช้ในพื้นที่ตรวจ

---

### Phase 2 — On-Demand (เดือน 2–3)
**เป้าหมาย:** กระดาษเป็น optional, ออกเมื่อมีคนขอเท่านั้น

- Store บันทึกใน WMS → กระดาษพิมพ์เฉพาะเมื่อ supplier ต้องการ DN copy
- QA ใช้ WMS เท่านั้น
- Trust signal ชัดเจน: status badge สีเขียว = QA pass

---

### Phase 3 — 100% Paperless (เดือน 3+)
**เป้าหมาย:** ไม่มีกระดาษในกระบวนการ GR→IQC

- Receiving: สแกน PO barcode → GR form auto-populate
- IQC: checklist digital → photo attach → approve ด้วย PIN/fingerprint
- Stock: location QR + WMS scan เท่านั้น

---

## Trust Signal Design

แทน stamp/signature ด้วย digital equivalents:

| Physical Signal | Digital Equivalent | Where |
|----------------|-------------------|-------|
| QA stamp บน DN | Status badge `IQC_PASS` + timestamp + user ID | WMS record |
| ลายเซ็น Store | Digital acceptance log (IP + device + timestamp) | Audit log |
| กระดาษ GRN เลขที่ | GRN number + QR code (พิมพ์ on-demand) | WMS |
| Sticker สีแดง "Hold" | Location flag `QA_HOLD` ใน system + physical tape | WMS + visual |

---

## Document ID Cross-Reference

ทุก GR record ต้องแสดง **4 เลขเอกสารพร้อมกัน:**

```
GR Record #GR-2026-05-001
├── Odoo PO:       PO/2026/05/0234
├── WMS GRN:       GRN-20260527-001
├── Supplier DN:   DN-ABC-250527-XX
└── Internal Ref:  [PO suffix 3 หลักท้าย]
```

**ทำไมต้องมีครบ 4:** พนักงานมักอ้างอิงด้วยเลขต่างกัน — Store ใช้ DN, Accounting ใช้ PO, QA ใช้ GRN cross-reference ทำให้ทุกคน trace ได้

---

## IQC Digital Checklist Template

```
GRN: [auto]          PO: [auto]
Supplier: [auto]     Item: [auto]
Qty Received: [ ]    Qty Inspected: [ ]

Inspection Items:
☐ 1. Visual inspection — ไม่มีรอยแตก/บิ่น/สี abnormal
☐ 2. Quantity verify — ตรงกับ DN
☐ 3. Part number verify — ตรงกับ PO
☐ 4. Packaging condition — sealed/intact
☐ 5. [Custom items per item category]

Result: ○ PASS  ○ FAIL  ○ CONDITIONAL PASS
Remarks: [free text]
Inspector: [user ID auto-fill]
Timestamp: [auto]
Photo: [attach]
```

---

## Metric ที่ต้อง Track

| Metric | Baseline (Paper) | Target (Paperless) |
|--------|-----------------|-------------------|
| GR processing time | 2–4 ชั่วโมง | < 30 นาที |
| IQC turnaround | 1–2 วัน | < 4 ชั่วโมง |
| Data entry error rate | ~5% | < 0.5% |
| Traceability query time | 20–30 นาที | < 1 นาที |

---

## Rollback Plan

ถ้าระบบ down ระหว่าง transition:

1. พิมพ์ manual GR form (PDF template เตรียมไว้)
2. กรอกด้วยมือ
3. เมื่อระบบกลับมา → backfill ข้อมูลจากกระดาษ
4. ระบุในหมายเหตุว่า "backfilled from paper — system downtime"

**สำคัญ:** ห้าม skip IQC แม้ระบบ down
