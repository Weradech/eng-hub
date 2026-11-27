---
title: "Scan File Metadata Before External Release"
date: 2026-11-27 18:00:00 +0700
categories: [NPI, Process]
tags: [release, document-control, metadata, verification]
description: "Inspect visible content, hidden metadata and embedded files, then verify the exact package available for download."
---

> **TL;DR** — Scan the files you plan to share, including images inside documents. After uploading, download the package through the delivery link and check its SHA-256 against the reviewed copy.

A drawing can look ready on screen while its illustration still carries a source filename in PNG metadata. If that image is embedded in a Word document, the filename can travel inside the document too. A review of the visible pages and a search of the document text cover only part of what the recipient gets.

## What to inspect

| What you are sending | What to open or scan |
|---|---|
| PNG and other images | Inspect the pixels and use a format-aware metadata reader. PNG can store text in `tEXt`, `zTXt` and `iTXt`, and Exif in `eXIf`. Check embedded images the same way. |
| Word, Excel and PowerPoint files | Run the document inspector on a release copy. Check document properties, comments, tracked changes, hidden sheets or slides where applicable, embedded media and objects, custom XML and external links. Review embedded files in their own formats. |
| PDF | Check document properties, including title, author and keywords, and any XMP metadata. Look for comments, attachments, hidden layers and searchable text behind page images. Reopen the exported PDF and review the rendered pages. |
| ZIP or other archive | List the entries, compare them with the release list and check for working copies or unapproved attachments. Open and inspect the contents before packing them. |

Look for personal details, local file paths, working filenames and confidential terms. Decide which values belong in the release, such as the approved document title and revision. Record any file the tools could not read so it can be reviewed before release.

## Check the copy the recipient gets

1. **Stage the delivery files.** Include the intended files on a release list. Review their visible content and run the format checks above. Route any change to a signed file through document control so the signed original is preserved.
2. **Correct and recheck.** Remove unintended information from the release copy and replace affected embedded assets. Reopen the result, verify the required content still renders and works, and rerun the inspection. If visible content needs redaction, use a redaction tool and verify the text or image data was removed.
3. **Record the reviewed bytes.** Record the file names and SHA-256 hashes. If delivering a ZIP snapshot, confirm its entry list and integrity, then record the archive hash as well.
4. **Download and compare.** Retrieve the delivered file or archive through the same link the recipient will use. If its hash differs, hold the handoff and compare the contents and metadata with the reviewed copy. A platform can repackage an Office file while the pages look the same. Recheck any changed copy before approving it for distribution.

A matching hash confirms the bytes that were retrieved match the reviewed bytes. Keep the inspection result and release approval with the hash so the team can trace what was checked and authorized.

For example, if the reviewed delivery is a ZIP containing a drawing PDF and a PNG, compare the hash of the downloaded ZIP with the reviewed ZIP. If the delivery link also offers the PDF as a separate download, check that copy against the reviewed PDF too.

## Further reading

- [PNG specification](https://www.w3.org/TR/png-3/)
- [Microsoft Office Document Inspector](https://support.microsoft.com/en-us/office/collab-files/remove-hidden-data-and-personal-information-by-inspecting-documents-presentations-or-workbooks)
- [Adobe Acrobat PDF sanitization](https://helpx.adobe.com/acrobat/desktop/protect-documents/redact-pdfs/sanitize.html)

## Related Posts

- [Engineering Release Package: Are These Files Ready to Build?]({% post_url 2026-09-09-engineering-release-package %})
- [Can We Release This Procedure Revision?]({% post_url 2026-11-22-release-a-controlled-document-revision %})
