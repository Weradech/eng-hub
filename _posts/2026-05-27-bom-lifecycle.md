---
title: "BOM Lifecycle — From RFQ Stage to Active Production in Odoo"
date: 2026-05-27 15:00:00 +0700
categories: [NPI, BOM]
tags: [bom, odoo, mrp, npi, rfq, erp]
---

> **TL;DR** — RFQ BOM ≠ Production BOM. Each stage has a different owner and format. Uploading the wrong BOM stage into ERP is a leading cause of MRP errors.

---

![A BOM moves through four stages — RFQ costing BOM, engineering eBOM, manufacturing mBOM, then the active BOM in Odoo — each with a different owner, store, and what gets locked]({{ "/assets/img/2026-05-27/bom-lifecycle.svg" | relative_url }})
_The same BOM wears four hats; load the wrong stage into ERP and MRP inherits the error._

## BOM Stages and Their Roles

```
RFQ BOM → Engineering BOM → Manufacturing BOM → Odoo Active BOM
(costing)  (design intent)   (process-aware)     (MRP source)
```

Each stage has a distinct owner and should never be used as a substitute for another.

---

## Stage 1 — RFQ BOM (Quotation Stage)

**Owner:** NPI / Sales Engineering  
**Stored in:** Excel file in project folder (not in ERP yet)

**Characteristics:**
- Includes pricing columns (unit price, total cost)
- May contain alternative parts
- Component specs may not be finalized
- Reference designators may be incomplete

**Used for:**
- Material cost calculation for quotation
- Sending to Procurement for price inquiries

**Never use for:** Sourcing a production order

---

## Stage 2 — Engineering BOM (eBOM)

**Owner:** Design Engineer / R&D  
**Stored in:** Altium / KiCad export, or PDM system

**Characteristics:**
- Complete reference designators (R1, C2, U3...)
- Clear part numbers with manufacturer PN
- No process information yet (reflow temp, paste type)
- May include DNI (Do Not Install) components

**Used for:**
- Design review
- DFM/DFA analysis
- ICT test list generation

---

## Stage 3 — Manufacturing BOM (mBOM)

**Owner:** Process / Manufacturing Engineer  
**Stored in:** ERP / MES (Odoo mrp.bom)

**Characteristics:**
- Derived from eBOM, with added process information:
  - Assembly side (Top / Bottom)
  - Solder type (Reflow / Wave / Hand)
  - Lot traceability requirements
- DNI components removed
- Approved substitutes registered in the system

**Used for:**
- Production order generation
- Material requisition
- MRP calculation

---

## Stage 4 — Active BOM in Odoo

**Owner:** Production / MRP Team  
**Stored in:** Odoo `mrp.bom` — only BOMs with an open PO and active status

```
Odoo mrp.bom:
  product_id    → Finished Good
  type          → manufacture
  bom_line_ids  → components (product + qty + uom)
```

**Key rules:**
- BOM in Odoo = **approved for production, with PO issued**
- RFQ-stage BOMs stay in Excel until PO is approved
- One Finished Good should have one Active BOM (multiple active BOMs confuse MRP)

---

## ECR Process (Engineering Change Request)

When a production BOM component needs to change:

```
1. Create ECR document
   - Reason (obsolescence / cost reduction / quality)
   - Old PN vs New PN
   - Affected boards / serial number range

2. Validation
   - EVT or qualification test (for critical components)
   - DFM check (if layout affected)

3. Update BOM in Odoo
   - Deactivate old BOM → create new revision
   - Set effective date / serial number range

4. Notify Production and QA
```

---

## Common BOM Mistakes

| Mistake | Consequence |
|---------|-------------|
| Upload RFQ BOM directly to Odoo | MRP orders wrong specs, incorrect cost |
| Multiple active BOMs for same product | MRP doesn't know which to use |
| Not updating Odoo BOM after ECR | Production uses obsolete components |
| Forgetting to remove DNIs from mBOM | Orders components that are never installed |
| Reference designators don't match layout | ICT fixture probes wrong locations |

---

## Summary Flow

```
NPI receives spec
    ↓
Create RFQ BOM (Excel) → use for costing
    ↓
Design produces eBOM (Altium export)
    ↓
Process Engineer creates mBOM + adds process info
    ↓
PO approved → upload to Odoo mrp.bom
    ↓
Production uses → MRP orders from Odoo
```

A clear BOM lifecycle = accurate MRP = correct purchasing = less waste.
