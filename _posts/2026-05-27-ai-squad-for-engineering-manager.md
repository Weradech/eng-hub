---
title: "AI Squad สำหรับ Engineering Manager — ทำงานคนเดียวได้เหมือนมีทีม 10 คน"
date: 2026-05-27 11:00:00 +0700
categories: [Tools, AI]
tags: [claude-code, ai, productivity, engineering-management, llm]
---

> **TL;DR** — Engineering Manager คนเดียวสามารถ run NPI + RFQ + BOM + SOP + System Integration ได้พร้อมกัน ด้วย AI agent squad ที่ถูก assign role ชัดเจน

---

## ปัญหาของ Engineering Manager คนเดียว

ใน SME ไทย Engineering Manager มักต้องทำทุกอย่างพร้อมกัน:
- Review BOM จากทีม procurement
- ตอบ RFQ ให้ลูกค้า
- ดู DFM/DFA กับ design team
- เขียน SOP/WI สำหรับ ISO audit
- Monitor production KPI

**ปัญหา**: งาน context-switch สูงมาก → output ช้า → deadline พลาด

---

## แนวคิด AI Squad

แทนที่จะใช้ AI เป็น "chatbot ตอบคำถาม" → ใช้เป็น **specialized agent** แต่ละตัวมี role ชัดเจน

```
Engineering Manager (คุณ)
        │
   ┌────┴────┐
   ▼         ▼
[BOM Agent] [RFQ Agent]   ← ทำ parallel
   │              │
[Design Agent] [Doc Agent] ← ส่งงานกันได้
```

---

## 4 Core Agents สำหรับ NPI/SE

### 1. BOM Agent — ตรวจ BOM + Cost Roll-up

**Prompt template:**
```
คุณคือ BOM Engineer ที่เชี่ยวชาญ PCBA
BOM ด้านล่างนี้: [วาง BOM]
ทำ:
1. ตรวจ reference designator ซ้ำ
2. Flag component ที่ไม่มี approved vendor
3. Roll-up material cost รวม OH 15%
Output: table + summary
```

**ใช้กับ:** Excel BOM, Altium BOM export, Odoo mrp.bom

---

### 2. RFQ Agent — เขียน quotation + technical scope

**Prompt template:**
```
คุณคือ NPI Engineer ที่ทำ quotation
Scope: [ใส่ spec/drawing summary]
คำนวณราคาตาม pattern:
- Material: [ราคาจาก BOM]
- Assembly: [จำนวน placements]
- OH: 15% on mfg
- Margin: 25%
- NRE: [tooling list]
Output: quotation format ภาษาอังกฤษ lump sum
```

---

### 3. DFM Agent — Review drawing สำหรับ manufacturability

**สิ่งที่ต้องให้:**
- PDF drawing หรือ screenshot
- Assembly process ที่จะใช้ (SMT/THT/Wave)
- Target annual volume

**Output ที่ได้:**
- DFM checklist ผ่าน/ไม่ผ่าน
- Recommended changes พร้อม rationale
- Risk ranking

---

### 4. Doc Agent — เขียน SOP/WI ตาม ISO structure

**Prompt template:**
```
เขียน Work Instruction สำหรับ [ชื่อ process]
ตาม ISO 9001:2015 structure:
- Header: Doc No / Rev / Effective Date / Process Owner
- Purpose (1 ย่อหน้า)
- Scope
- Steps (numbered, action-oriented)
- Quality checkpoint ทุก critical step
- Related documents
ภาษา: ไทย
```

---

## Setup จริงที่ใช้อยู่

### Claude Code (Primary)
- **Model**: Claude Sonnet 4.6
- **Use case**: Complex analysis, BOM review, SOP เขียน, code
- **Cost**: ~$3/hr heavy use

### Gemini CLI (Secondary)
- **Use case**: Browser-bound tasks, NotebookLM upload, Excel reading
- **Strength**: Google Workspace integration

### NotebookLM (Knowledge Base)
- Upload SOP + Drawing + BOM เก่าๆ ทั้งหมด
- Query ด้วย natural language ว่า "เคยทำ project แบบนี้ไหม?"
- ช่วย recall ได้เร็วมาก

---

## Workflow ตัวอย่าง — RFQ ใหม่ใน 30 นาที

```
1. รับ spec จากลูกค้า (5 นาที)
   ↓
2. BOM Agent: extract component list จาก spec (5 นาที)
   ↓
3. BOM Agent: query ราคาจาก Odoo/supplier portal (5 นาที)
   ↓
4. RFQ Agent: calculate + generate quotation draft (10 นาที)
   ↓
5. Review + approve + send (5 นาที)
```

เดิมใช้เวลา 1–2 วัน → เหลือ 30 นาที

---

## Tips จากประสบการณ์

| Do | Don't |
|----|-------|
| ให้ context ชัดเจน (เป็น engineer ระดับไหน?) | ถามกว้างๆ "ช่วยทำ BOM หน่อย" |
| ตรวจ output ทุกครั้ง โดยเฉพาะตัวเลข | Blind trust output |
| ใช้ structured prompt (role + task + output format) | Free-form paragraph |
| ให้ sample output ที่ต้องการ | อธิบาย output แค่คำ |

---

## ROI จริง

| งาน | เดิม | ด้วย AI |
|-----|------|---------|
| PCBA Quotation | 1–2 วัน | 30–60 นาที |
| SOP เขียนใหม่ | 3–5 วัน | 2–4 ชั่วโมง |
| BOM Review | 2–3 ชั่วโมง | 15–30 นาที |
| RCA Report | 1 วัน | 2–3 ชั่วโมง |

---

> จุดสำคัญ: AI ไม่ได้แทน engineering judgment — AI ลด **administrative overhead** ให้คุณมีเวลา focus ที่ decision-making จริงๆ
