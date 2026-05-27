---
title: "BOM Lifecycle — จาก RFQ Stage ถึง Active Production ใน Odoo"
date: 2026-05-27 15:00:00 +0700
categories: [NPI, BOM]
tags: [bom, odoo, mrp, npi, rfq, erp]
---

> **TL;DR** — BOM มีหลาย stage ที่แตกต่างกันมาก RFQ BOM ≠ Production BOM การใส่ BOM ผิด stage เข้า ERP คือต้นเหตุของ MRP เสีย

---

## BOM แต่ละ Stage ทำอะไร?

```
RFQ BOM → Engineering BOM → Manufacturing BOM → Odoo Active BOM
   (costing)    (design intent)   (process-aware)    (MRP source)
```

ทุก stage มีเจ้าของและ format ต่างกัน ห้ามใช้แทนกัน

---

## Stage 1 — RFQ BOM (Quotation Stage)

**เจ้าของ:** NPI / Sales Engineering  
**เก็บที่:** Excel file ใน project folder (ยังไม่ขึ้น ERP)

**ลักษณะ:**
- มี pricing column (unit price, total cost)
- อาจมี alternative part
- component spec อาจยังไม่ finalized
- ไม่มี reference designator ครบ

**ใช้สำหรับ:**
- คำนวณ material cost สำหรับ quotation
- ส่งให้ Procurement สอบถามราคา

**ห้ามนำไปใช้:** เป็น source สำหรับ production order

---

## Stage 2 — Engineering BOM (eBOM)

**เจ้าของ:** Design Engineer / R&D  
**เก็บที่:** Altium / KiCad export, หรือ PDM system

**ลักษณะ:**
- มี Reference Designator ครบ (R1, C2, U3...)
- Part number ชัดเจน พร้อม manufacturer PN
- ยังไม่มี process information (reflow temp, paste type)
- อาจมี DNI (Do Not Install) components

**ใช้สำหรับ:**
- Design review
- DFM/DFA analysis
- ICT test list generation

---

## Stage 3 — Manufacturing BOM (mBOM)

**เจ้าของ:** Process / Manufacturing Engineer  
**เก็บที่:** ERP / MES (Odoo mrp.bom)

**ลักษณะ:**
- Derived จาก eBOM แต่เพิ่ม process info:
  - Assembly side (Top/Bottom)
  - Solder type (Reflow/Wave/Hand)
  - Lot traceability requirement
- DNI ถูก remove ออกแล้ว
- Substitute part approved อยู่ใน system

**ใช้สำหรับ:**
- Production order generation
- Material requisition
- MRP calculation

---

## Stage 4 — Active BOM ใน Odoo

**เจ้าของ:** Production/MRP Team  
**เก็บที่:** Odoo `mrp.bom` (เฉพาะ BOM ที่เปิด PO แล้ว + active)

```
Odoo mrp.bom:
  product_id  → Finished Good
  type        → manufacture
  bom_line_ids → components (product + qty + uom)
```

**Rule สำคัญ:**
- BOM ใน Odoo = **ผ่าน PO และ approved สำหรับ production แล้วเท่านั้น**
- RFQ stage BOM ยังอยู่ใน Excel → ยังไม่ขึ้น Odoo
- หนึ่ง Finished Good ควรมี Active BOM เดียว (ถ้ามีหลายตัว MRP งง)

---

## BOM ECR (Engineering Change Request)

เมื่อต้องเปลี่ยน component ใน production BOM:

```
1. สร้าง ECR document
   - เหตุผล (obsolescence / cost reduction / quality)
   - Old PN vs New PN
   - Affected boards / serial number range

2. Validation
   - EVT หรือ qualification test (ถ้า critical component)
   - DFM check (ถ้าแก้ layout)

3. Update BOM
   - Odoo: deactivate old BOM → create new revision
   - ระบุ effective date / serial number range

4. Notify Production + QA
```

---

## Common BOM Mistakes

| Mistake | ผลที่ตามมา |
|---------|-----------|
| ใส่ RFQ BOM ขึ้น Odoo โดยตรง | MRP สั่งซื้อ spec ผิด, cost ผิด |
| มี BOM หลายตัว active พร้อมกัน | MRP ไม่รู้จะใช้ตัวไหน |
| ไม่ update Odoo BOM หลัง ECR | Production ใช้ component เก่า |
| ลืม remove DNI จาก mBOM | สั่งซื้อ component ที่ไม่ได้ใส่บอร์ด |
| Reference designator ไม่ตรงกับ layout | ICT fixture test ผิดจุด |

---

## สรุป Flow

```
NPI รับ spec
    ↓
สร้าง RFQ BOM (Excel) → ใช้คิดราคา
    ↓
Design ออก eBOM (Altium export)
    ↓
Process Engineer สร้าง mBOM + add process info
    ↓
PO approved → upload ขึ้น Odoo mrp.bom
    ↓
Production ใช้ → MRP สั่งซื้อจาก Odoo
```

BOM lifecycle ชัดเจน = MRP แม่นยำ = สั่งของตรง = ลด waste ค่ะ
