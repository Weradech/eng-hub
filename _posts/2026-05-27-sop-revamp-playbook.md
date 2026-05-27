---
title: "SOP Revamp Playbook — 13 หลักการเขียน SOP ที่ใช้งานได้จริง"
date: 2026-05-27 14:00:00 +0700
categories: [ISO, Documentation]
tags: [sop, iso9001, work-instruction, process-documentation, quality]
---

> **TL;DR** — SOP ที่ดีไม่ใช่ SOP ที่ยาวที่สุด แต่คือ SOP ที่พนักงานใหม่อ่านแล้วทำงานได้ทันที 13 หลักนี้มาจากการ revamp เอกสาร EN department จริง

---

## ปัญหาของ SOP แบบเก่า

SOP ส่วนใหญ่ใน SME ไทยมีปัญหาเหมือนกัน:
- เขียนครั้งเดียวแล้วไม่อัพเดท
- ยาวเกิน → ไม่มีใครอ่าน
- ใช้ภาษาคลุมเครือ ("ควร", "อาจ", "โดยประมาณ")
- ไม่มี Process Owner ชัดเจน → ไม่มีใครรับผิดชอบ
- เขียนเพื่อ auditor ไม่ใช่เพื่อผู้ใช้งาน

---

## ISO Document Hierarchy

```
Tier 1 — Quality Manual (P)
    ↓ อ้างอิง
Tier 2 — Procedure (P) — "ทำอะไร ทำไม ใคร"
    ↓ อ้างอิง
Tier 3 — Work Instruction (WI) — "ทำอย่างไร ขั้นตอนละเอียด"
    ↓ ใช้
Tier 4 — Form / Record (FM) — บันทึกผล
```

**ความผิดพลาดที่พบบ่อย:** เขียน Procedure ละเอียดเหมือน WI → ซ้ำกัน ดูแลยาก

---

## 13 หลักการ

### 1. Document Hierarchy ต้องชัดก่อนเขียน
กำหนดก่อนว่าสิ่งที่เขียนคือ P หรือ WI ไม่ใช่ผสมกัน

### 2. Header ต้องมีครบ 6 field
```
Doc No: EN-P-001        Rev: 02
Title:  NPI Gate Review Process
Effective Date: 2026-05-01
Process Owner: NPI Manager
Approved by: MR (Management Representative)
```
**MR = Approver เสมอ** ไม่ใช่ Department Head

### 3. Process Owner ≠ ผู้เขียน
Process Owner คือคนที่:
- รับผิดชอบ output ของ process
- ต้องทบทวน SOP เมื่อ process เปลี่ยน
- ถูก audit ถาม

### 4. Ownership ตาม Business Function
แบ่ง ownership ตาม department จริง:

| Function | Process Owner |
|----------|--------------|
| NPI Gate | NPI Manager |
| Drawing Release | R&D Lead |
| Production SOP | Production Manager |
| IQC Procedure | QA Manager |

### 5. ใช้ภาษา Action-Oriented
| ❌ คลุมเครือ | ✅ ชัดเจน |
|------------|---------|
| ควรตรวจสอบ | ตรวจสอบ [รายการ X] ด้วย [เครื่องมือ Y] |
| โดยประมาณ | ± 0.5mm |
| บางครั้ง | ทุกครั้งที่ [เงื่อนไข Z] |

### 6. Gate Adaptation — ปรับ checkpoint ตาม risk
ไม่ใช่ทุก step ต้องมี checkpoint เดียวกัน:
- High risk step → checkpoint บังคับ + sign-off
- Routine step → self-check พอ

### 7. Business Context Integration
SOP ต้องอธิบาย **ทำไม** ไม่ใช่แค่ **อะไร**:

```
❌ "ตรวจสอบ BOM ก่อน PO"
✅ "ตรวจสอบ BOM ก่อน PO เพราะ supplier ไม่รับ
   claim หาก part number ผิด — ทำให้ cost เพิ่มและ
   delivery ล่าช้า"
```

### 8. Compliance แบบ Selective
ISO require อะไรต้องมี แต่ที่ ISO ไม่ได้ require ไม่ต้องทำ:
- ✅ Risk assessment ต้องมี (ISO 9001:2015 clause 6.1)
- ❌ ไม่ต้องมี approval signature ทุก revision ถ้า e-approval ได้

### 9. ตัด Bloat ออก
Section ที่มักไม่มีคุณค่า:
- "วัตถุประสงค์" ยาว 3 ย่อหน้า → 1 ประโยคพอ
- "นิยาม" ที่ทุกคนรู้อยู่แล้ว → ตัดออก
- "References" ที่ไม่มีใครเปิดดู → เหลือแค่ที่ใช้จริง

### 10. Numbering Hygiene
```
✅ ใช้: 4.1, 4.2, 4.2.1
❌ หลีกเลี่ยง: 4.1.1.1.1 (ลึกเกิน 3 level)
❌ หลีกเลี่ยง: Section A, Section B (ผสมกับตัวเลขแล้วงง)
```

### 11. หลวมๆ ได้ที่ Procedure ระดับ P
Procedure (P) เขียนกว้างได้ — ความละเอียดอยู่ที่ WI
อย่าพยายามให้ P ครอบคลุมทุก edge case

### 12. Auto-scan ก่อน Release
ก่อน release document ทุกครั้ง:
- [ ] Doc number ซ้ำกับที่มีอยู่ไหม?
- [ ] Revision ถูกต้องไหม?
- [ ] Process Owner เซ็นหรือ approve แล้วหรือยัง?
- [ ] Related documents ที่อ้างถึงยัง active ไหม?

### 13. Lessons Learned ต้องเป็น Living Document
SOP ที่ดีต้องอัพเดทเมื่อ:
- มี non-conformance จาก process นั้น
- Process เปลี่ยนแปลงในทางปฏิบัติ
- Auditor feedback
- พนักงานใหม่ทำตามแล้ว confused

**Target: review ทุก 12 เดือน หรือเมื่อมี trigger**

---

## Work Instruction Template

```markdown
Doc No: [EN-WI-XXX]    Rev: [00]    Date: [YYYY-MM-DD]
Title:  [ชื่อ WI]
Process Owner: [ชื่อ + ตำแหน่ง]
Approved by: [MR]

## 1. Purpose (1 ประโยค)
WI นี้อธิบายวิธี [action] เพื่อ [outcome]

## 2. Scope
ใช้กับ [ระบุขอบเขต]

## 3. Tools / Materials Required
- [รายการ]

## 4. Steps
1. [Action verb] + [object] + [criterion]
2. ✓ Checkpoint: [verify อะไร ด้วยวิธีไหน]
3. ...

## 5. Related Documents
- [Doc No]: [Title]

## 6. Record
บันทึกผลใน [Form No]: [Form Name]
```

---

## สรุป

SOP ที่ดี = **สั้น + ชัด + ใช้งานได้ + มี owner + อัพเดทได้**

ถ้า SOP ยาวเกิน 3 หน้าสำหรับ task เดียว → แบ่งเป็นหลาย WI แทนค่ะ
