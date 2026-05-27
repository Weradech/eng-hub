---
title: "Financial Core vs Operational Core — Mental Model สำหรับ System Integration"
date: 2026-05-27 16:00:00 +0700
categories: [System, Architecture]
tags: [erp, wms, mrp, odoo, system-integration, architecture]
---

> **TL;DR** — ERP (Odoo) = Book of Record สำหรับเงิน; ระบบปฏิบัติการ (WMS/MRP/MES) = System of Engagement สำหรับงาน ทั้งสองต้องคุยกันแต่ห้าม merge เป็นระบบเดียว

---

## ปัญหาที่เกิดจากไม่แยก Layer

หลายบริษัท SME พยายามให้ ERP ทำทุกอย่าง:
- บันทึก GR ใน ERP → slow, ไม่ real-time
- พนักงาน warehouse ใช้ ERP โดยตรง → error สูง
- หรือบางทีกลับกัน: ระบบ warehouse ทำเอง → ERP accounting ไม่รู้

ผลคือ **ข้อมูลสองระบบไม่ตรงกัน** = ไม่รู้จะเชื่ออะไร

---

## Two-Layer Mental Model

```
┌─────────────────────────────────────────┐
│         FINANCIAL CORE (ERP)            │
│  Odoo — Book of Record                  │
│                                         │
│  PO / Bill / Payment / GL / Invoice     │
│  "ตัวเลขที่ใช้ยื่นภาษีและรายงานผู้บริหาร"      │
└──────────────────┬──────────────────────┘
                   │ sync (scheduled / event)
┌──────────────────▼──────────────────────┐
│       OPERATIONAL CORE                  │
│  WMS / MRP / MES / CRM / AP Tracker    │
│  System of Engagement                   │
│                                         │
│  GR / GI / Production / IQC / Delivery  │
│  "สิ่งที่ทีมทำจริงๆ ทุกวัน"                 │
└─────────────────────────────────────────┘
```

---

## Financial Core — ERP (Odoo)

**Authority on:**
- PO (Purchase Order) — สัญญากับ supplier
- Vendor Bill — ใบแจ้งหนี้
- Payment — การจ่ายเงิน
- General Ledger — บัญชีแยกประเภท
- Customer Invoice

**Rules:**
- ทุก transaction ที่กระทบเงิน → ต้องผ่าน ERP
- ERP คือ single source of truth สำหรับตัวเลขการเงิน
- ห้ามแก้ตัวเลขการเงินนอก ERP

**Analogy:** ธนาคาร — ทุกการเคลื่อนไหวเงินต้อง record ที่นี่

---

## Operational Core — Systems of Engagement

**Authority on:**
- Inventory movement (GR/GI/Transfer)
- Production status (MO start/complete)
- Quality inspection (IQC pass/fail)
- Customer interaction (CRM lead/quote)

**Rules:**
- Optimize for speed + usability + real-time
- User-friendly UI → ลด error
- Push summary ไปหา ERP (ไม่ใช่ real-time sync ทุก transaction)

**Analogy:** หน้าร้าน — ลูกค้าและพนักงานทำงานที่นี่ แต่สิ้นวันต้องส่งยอดเข้าระบบบัญชี

---

## Integration Pattern

### Sync Direction

```
Operational → Financial (Push summary)
  WMS GR complete → Odoo stock.move + vendor bill matching

Financial → Operational (Pull master data)
  Odoo: Item master, Supplier, Customer, Analytic account
  → ดึงเข้า WMS/MRP ก่อนใช้งาน
```

### Two-Status Pattern

transaction เดียวมีได้ 2 status:

| Status Type | Owner | ตัวอย่าง |
|-------------|-------|---------|
| Operational | WMS | `received` → `iqc_pass` → `in_stock` |
| Financial | Odoo | `draft` → `posted` → `paid` |

ทั้งสองไม่ต้องเป็น 1:1 เสมอ เช่น:
- WMS: `iqc_pass` (ของเข้าคลังแล้ว)
- Odoo: Bill ยัง `draft` (รอ Accounting post)

---

## Common Integration Anti-patterns

| Anti-pattern | ปัญหา |
|-------------|-------|
| Read Odoo stock ตรงจาก WMS ทุก transaction | Odoo load สูง, latency สูง |
| WMS เขียน Odoo GL โดยตรง | ข้าม accounting rules, audit fail |
| Real-time sync ทุก field | network dependency สูง — WMS ดับเมื่อ Odoo down |
| ไม่มี sync log | ไม่รู้ว่า transaction ไหน push ไป Odoo แล้ว |

---

## Outbox Pattern สำหรับ Reliability

```
WMS transaction complete
    ↓
Write to outbox table (local DB)
    ↓
Background job push ไป Odoo ทุก 15 นาที
    ↓
Mark outbox as sent หลัง Odoo confirm
    ↓
ถ้า fail → retry with exponential backoff
```

**ข้อดี:** WMS ไม่ขึ้นกับ Odoo availability → ทำงานได้แม้ Odoo down

---

## สรุป

> ถามตัวเองก่อนทุก design decision:
> - "transaction นี้กระทบเงินไหม?" → ต้องผ่าน ERP
> - "transaction นี้คือการทำงานจริงของทีมไหม?" → อยู่ใน Operational system
> - "ทั้งสองระบบต้องรู้ไหม?" → sync ด้วย outbox pattern

แยก layer ชัดเจน = scale ได้ + maintain ง่าย + audit ผ่านค่ะ
