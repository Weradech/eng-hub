# Engineering release package templates

These are blank working records accompanying the Engineering Release Package
article in Syntech Engineering Hub. No customer design or release approval is included.

## Start here

1. Copy `release-checklist.md` and `release-manifest.md` into a new build package.
2. Define the product, variant, authorized activity sought and applicable order/lot.
3. Record required deliverables and justified N/A items. Inventory the actual files.
4. Complete the checks with traceable evidence, including images for DFx findings.
5. Obtain the actual scoped decision under the organization's existing authority rules.
6. Calculate final hashes, issue the reviewed package and record recipient acknowledgement.

Use `release-manifest.json` instead of the Markdown manifest when structured data
is useful. If maintaining both, designate one authoritative version and reconcile
the other before issue. The JSON is a data template, not a schema or validation program.
Parsing successfully does not establish completeness or release readiness.

Keep `DRAFT` and `PENDING` until the actual record supports changing them. Empty
arrays mean no entries have been recorded; they do not mean checks passed or that
there are no open issues. Leave decision/approval fields empty until evidenced.

## JSON field conventions

- `template_version`: version of this template structure, separate from the package revision.
- `record_status`: suggested lifecycle values are `DRAFT`, `ISSUED`, `SUPERSEDED`.
- `build_scope`: activity being reviewed, such as bare PCB fabrication or PCBA assembly.
- `effectivity`: applicable order, lot, serial range, quantity and quantity basis.
- `source_baseline`: controlled design revision, source commit or export record.
- `authoritative_manifest`: identity/path of the record governing the package.
- Dates: use ISO 8601, including a timezone offset for timestamps.
- Paths: relative to the package root. Evidence may be a controlled record reference
  or an accessible URL when the source is outside the package.

Add one `deliverables` object for each role being considered:

```json
{
  "role": "<deliverable role>",
  "applicability": "<REQUIRED or N/A>",
  "requirement_source_or_na_reason": "<record reference and reason>",
  "file_paths": [],
  "open_item": null
}
```

Add one `files` object for every controlled file being issued:

```json
{
  "path": "<package-relative path>",
  "role": "<file purpose>",
  "document_revision": "<actual document revision>",
  "sha256": "<64 hexadecimal characters calculated from final file bytes>",
  "source_baseline": "<controlled source or export record>",
  "review_evidence": "<check record and location>"
}
```

Angle-bracket values above are instructions and must be replaced. They are not
valid completed-record values. For an archive, link a member inventory from its
review evidence. Exclude the manifest itself from its own hash inventory.

Add `review.open_items` entries with `id`, `description`, `affected_activity`,
`owner`, `closure_trigger`, `required_evidence`, and `closure_evidence` fields.
Leave closure evidence empty until the action is verified.

`review.decision` starts as `PENDING`. Suggested decisions are `HOLD`,
`GO WITH CONDITIONS`, and `GO`, subject to existing organizational policy.
For a conditional decision, add `conditions` entries identifying the restriction,
permitted activity/quantity/lot, owner, closure trigger and acceptance evidence.
Record the person, authority, date and evidence of the decision separately.

## Hash and issue handling

For a single finalized file in PowerShell:

```powershell
Get-FileHash -LiteralPath '.\fabrication\board-fabrication.zip' -Algorithm SHA256
```

Replace that illustrative path with an actual package file. Copy the resulting
hash into its manifest entry. After creating the final delivery archive, record
its hash in the external transmittal; do not reopen the archive to insert its own hash.
Leave the archive-hash field empty in a manifest contained inside that archive.
Populate it only in an external issue record or detached manifest copy.

Preserve previously issued packages. A subsequent file change requires impact
review, updated hashes and a new package revision with a supersession reference.

## Reference

KiCad 9 PCB Editor documentation explains placement export choices, including
DNP/through-hole exclusions, coordinate origin and bottom-side handling:
https://docs.kicad.org/9.0/en/pcbnew/pcbnew.html#_component_placement_files

Confirm the actual tool version and receiving manufacturer's conventions for each build.
