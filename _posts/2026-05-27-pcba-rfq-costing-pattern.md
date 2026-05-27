---
title: "PCBA RFQ Costing Pattern — วิธีคิดราคา PCBA แบบมืออาชีพ"
date: 2026-05-27 09:00:00 +0700
categories: [NPI, RFQ]
tags: [pcba, costing, rfq, bom, npi, ems]
---

> **TL;DR** — 6 ขั้นตอนคิดราคา PCBA ตั้งแต่ BOM จนถึง Quotation: Material → Assembly → OH → NRE → Margin → Final Price พร้อม rule-of-thumb ที่ใช้ได้จริง

---

## ทำไมต้อง Costing ให้ถูกต้อง?

ใน PCBA manufacturing ข้อผิดพลาดในการ costing เพียง 5–10% บน material cost สามารถทำให้ project ขาดทุนได้ทันที เพราะ margin ของ EMS/OEM มักอยู่ที่ 15–25% เท่านั้น

---

## 6-Step Costing Pattern

### Step 1 — BOM Cost Roll-up

รวม cost ทุก component จาก BOM:

```
Total Material Cost = Σ (Unit Price × Qty per board)
```

**สิ่งที่ต้องระวัง:**
- ใช้ราคา **MOQ-adjusted** ไม่ใช่ unit price จาก datasheet
- รวม **DNI (Do Not Install)** ออกก่อนคำนวณ
- ราคา PCB bare board แยกเป็น line item เสมอ

---

### Step 2 — Assembly Cost

```
Assembly Cost = (Placement Count × Rate per placement) + Machine Setup
```

**Rate reference (Thai EMS, 2026):**
| Type | Rate |
|------|------|
| SMD 0402+ | ฿0.30–0.50/placement |
| SMD 0201 | ฿0.80–1.20/placement |
| Through-hole | ฿1.50–3.00/placement |
| Machine Setup | ฿17,500/lot |

---

### Step 3 — Overhead (OH)

> **Rule: OH = 15% applied on Manufacturing Costs เท่านั้น**

```
OH = (Material Cost + Assembly Cost) × 15%
```

⚠️ **ข้อผิดพลาดที่พบบ่อย** — บาง engineer คำนวณ OH บน total รวม NRE ด้วย ซึ่งผิด NRE ถือเป็น one-time cost แยกออกมาก่อนคำนวณ OH

---

### Step 4 — NRE (Non-Recurring Engineering)

NRE รวมทุก one-time cost:

| รายการ | หมายเหตุ |
|--------|----------|
| Stencil (SMT) | ฿3,500–8,000/ชิ้น |
| Fixture (ICT/FCT) | ขึ้นอยู่กับ complexity |
| Tooling (Wave solder jig) | |
| RE Fee (Reverse Engineering) | เฉพาะกรณีไม่มี source files |
| Engineering hours | rate × hours |

**สำหรับ Laser (shared asset):**
```
Laser NRE = ฿0 (shared)
Laser per unit = ฿0.50/unit
```

---

### Step 5 — Margin

```
Selling Price = (Material + Assembly + OH) × (1 + Margin%)
             + NRE (lump sum แยก)
```

**Margin guideline:**
- Prototype/EVT: 25–35%
- Mass production: 15–20%
- Strategic account: negotiable

---

### Step 6 — Quotation Format

> **Golden Rule: Quotation ส่งลูกค้า = Lump sum เท่านั้น ห้ามแสดง OH/Margin**

```
Unit Price: ฿XXX.XX / board
NRE: ฿XX,XXX (one-time)
MOQ: XXX pcs
Lead time: X weeks
Validity: 30 days
```

---

## Case Study — CDH1131A/LDU M

| Item | Value |
|------|-------|
| BOM Lines | 21 components |
| Total Placements | 58 |
| Material Cost | ฿94.82/board |
| Assembly | ฿28.46/board |
| OH (15%) | ฿18.49 |
| Unit Price (25% margin) | ฿ ~190/board |
| NRE Stencil | ฿5,500 |

---

## Script สำหรับ Auto-calculate

ถ้ามี BOM เป็น Excel/.csv สามารถใช้ Python script calculate ได้อัตโนมัติ:

```python
# สร้าง BOM dataframe จาก Excel
df = pd.read_excel("bom.xlsx")
material_cost = (df["unit_price"] * df["qty"]).sum()
assembly_cost = df["qty"].sum() * 0.40 + 17500/qty_per_lot
oh = (material_cost + assembly_cost) * 0.15
selling_price = (material_cost + assembly_cost + oh) / (1 - margin)
```

---

## สรุป

```
Final Price = [(Material + Assembly) × 1.15] / (1 - Margin%) + NRE/lot
```

Pattern นี้ใช้ได้กับทุก PCBA ตั้งแต่ proto 5 ชิ้นจนถึง production 10,000 ชิ้น ปรับแค่ margin และ MOQ-adjusted material price ค่ะ
