---
title: "Retemplate Controlled Forms Without Changing Their Content"
date: 2026-11-27 18:00:00 +0700
categories: [ISO, Documentation]
tags: [document-control, forms, templates, verification]
description: "Use one reference template, compare the editable body and inspect the exported PDF before handing a reformatted form to document control."
---

A form can pass a text comparison and still show two headers in its PDF. This can happen when a title block embedded in the document body survives the move to a new page header. A reliable template change checks both the editable document and the copy people will use.

## Set the scope before formatting

1. **Choose one reference per form type.** Record the reference revision, page size, orientation and margins. Identify any wide tables that need to stay in landscape.
2. **Mark what can move.** Identify the old title block, page-number fields and other elements the new header or footer will replace. Record the fields, instructions, tables and signature boxes that belong to the form body. Keep signed originals in document control and work on copies.

## Compare the source and the exported PDF

3. **Compare the editable body.** Compare paragraphs, table cells and fields with the baseline. Review every unexpected difference. Check any objects or fields the comparison tool cannot read separately.
4. **Inspect the PDF at reading size and in close-up.** Check that the title block appears once, fields are readable, page numbers make sense and tables and signature boxes stay intact. Review page breaks where content moved. If pagination changes, check that each affected page still carries the information it should.
{: start="3"}

For example, if a form has a title block in its first table and the new template already supplies one in the page header, remove only the identified legacy block. Then compare the remaining table content with the baseline and open the PDF to verify the resulting header.

Give the reviewer the reference revision, the body comparison and any layout exceptions. Keep the reformatted copy in draft until document control records approval.

**Related post:** [Can We Release This Procedure Revision?]({% post_url 2026-11-22-release-a-controlled-document-revision %})
