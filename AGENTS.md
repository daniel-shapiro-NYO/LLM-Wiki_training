
# Wiki Agent Instructions

## Session Model

Each chat session handles ONE ingest or ONE query. Start a fresh chat for
each operation. All state lives in the wiki files — never in chat history.

## If a file is attached to the chat message

1. Move it to `raw/` using a terminal command
2. Then proceed with the normal ingest flow

## Your Role

You maintain a persistent knowledge wiki. You own everything inside `wiki/`.
You never modify files in `raw/`. When asked to ingest a file, you handle all
conversion and extraction yourself using terminal commands as needed.

---

## Tagging Dimensions

Every source page carries two independent tags: **bucket** and **workstream**.

- **Bucket** = provenance — who authored the document. Auto-inferred from
  document content. This is about where the material came from.
- **Workstream** = engagement purpose — what consulting task this feeds.
  Set by the user's ingest command. This is about what the material is used for.

These are orthogonal: an external research report and a client capabilities deck
can both live inside the same workstream. A single workstream can draw from
multiple buckets.

---

## Buckets

Buckets are fixed provenance labels. **Edit the table below to match your
engagement before ingesting your first document.** The names you choose here
become the allowed values in every source page's `bucket:` frontmatter field.

| Bucket name | Meaning |
|---|---|
| `client-internal` | Authored by or on behalf of the client (pitch decks, SOW responses, capabilities docs, QBRs, internal ops reviews, meeting notes prepared by the client) |
| `external-research` | External material not authored by the client (analyst reports, academic papers, industry benchmarks, third-party case studies, competitor filings) |

> **To add a new bucket:** add a row to this table with a lowercase-hyphenated
> name and a one-sentence description. Do not invent bucket names on the fly
> during ingestion — define them here first. If authorship is genuinely
> ambiguous, set `bucket: unknown` and flag for human review.

### Auto-detecting the bucket on ingest

Scan the document title, header, cover page, and first substantive section for
authorship signals. Apply the first matching rule:

1. Document title, cover, or header clearly identifies an external author
   (analyst firm, academic institution, competitor, industry association) →
   `external-research`
2. Document is authored by or prepared on behalf of the client (pitch decks,
   SOW responses, capability decks, QBR pre-reads, internal ops boards,
   clarification memos, follow-up Q&A) → `client-internal`
3. Authorship cannot be clearly determined → set `bucket: unknown` and flag for
   human review in the ingest log entry.

The user can always override by stating the bucket explicitly (e.g.,
"Ingest this as client-internal" or "Ingest this as external-research").
An explicit user instruction always takes precedence over auto-detection.

When new bucket names are needed, the user will provide them. Do not invent
new bucket names — flag unknown authorship with `bucket: unknown` instead.

---

## Workstreams

A workstream tags sources by their consulting engagement purpose, preventing
cross-contamination in deliverables.

### Detecting the workstream on ingest

- **"Ingest this"** (no workstream named) → use `workstream: general`
- **"Ingest this for Workstream XYZ"** → use `workstream: xyz`
  (lowercase-hyphenated, e.g. `workstream: market-sizing`)
- The workstream value goes into the source page frontmatter AND is added to
  the `workstreams` list on every entity/concept page created or updated during
  that ingest. This is the workstream name.

### Workstream scoping during queries

- If the user's request implies a specific deliverable or context (e.g.,
  "draft an RFP response", "summarize the competitive landscape"), prefer
  sources whose `workstream` matches that context plus `general`.
- If the user explicitly names a workstream in the query (e.g., "using only
  the external research materials"), restrict retrieval strictly to that
  workstream.
- If no workstream context is detectable, use all sources but note which
  workstream and bucket each piece of information originates from.
- Never silently mix workstream-specific evidence into a deliverable scoped
  to a different workstream. If a cross-workstream fact seems relevant,
  surface it explicitly as an out-of-scope observation.
- Bucket can be used as an additional filter: "using only internal client
  materials" → scope to `bucket: client-internal`.

---

## On Every Ingest, You Will

1. Convert the source file to readable text if needed (run pandoc or pymupdf
   via terminal — install dependencies yourself if missing)
2. Auto-detect the bucket from document content (see Buckets section above)
3. Detect the workstream from the user's ingest command (see Workstreams above)
4. Write a summary page to `wiki/sources/<document-name>.md`
5. Create or update a page in `wiki/entities/` for every named
   company, product, person, or organization mentioned
6. Create or update a page in `wiki/concepts/` for every key theme,
   framework, or mechanism
7. Update `wiki/index.md`
8. Append a timestamped entry to `wiki/log.md`

---

## Page Format

### Source Page (`wiki/sources/<document-name>.md`)

```
---
title:
source_file:
date_ingested:
type: [paper | deck | report | transcript | other]
bucket: [client-internal | external-research | unknown]
workstream: [general | <workstream-name>]
---

# [Title]

## Summary
3–5 paragraph synthesis of the document's key arguments and findings.

## Key Data Points
Specific numbers, dates, statistics, quotes — preserved verbatim.

## Entities Referenced
- [[Entity Name]] — one-line role in this document

## Concepts Referenced
- [[Concept Name]] — one-line relevance

## Contradictions / Open Questions
Anything that conflicts with existing wiki pages, or raises new questions.
```

### Entity Page (`wiki/entities/<name>.md`)

```
---
title:
type: [company | product | person | trial | institution]
sources: []
workstreams: [general]
---

# [Name]

## Overview

## Key Facts

## Source Appearances
- [[source-document-name]] — one-line context

## Related
- [[linked page]] — relationship
```

### Concept Page (`wiki/concepts/<name>.md`)

Same structure as entity pages. `type:` should be one of:
`theme | framework | mechanism | market`

Frontmatter must include `workstreams: []` — same rule as entity pages.

---

## Conventions

- All internal links use `[[double bracket]]` Obsidian format
- Filenames are lowercase-hyphenated: `cardiovascular-risk.md`
- Never delete pages — mark deprecated ones with `deprecated: true` in frontmatter
- When two sources contradict each other, flag it in both source pages
  AND the relevant entity/concept page
- Preserve all specific data points verbatim — never paraphrase numbers
- Workstream names are lowercase-hyphenated and stable — once set, never
  rename a workstream name (it would break frontmatter across many pages)

---

## index.md Format

Keep a Markdown table for each category: Sources, Entities, Concepts.

- **Sources table row:** page link | one-line description | bucket | workstream | date added
- **Entities table row:** page link | type | one-line description
- **Concepts table row:** page link | one-line description | date added

---

## log.md Format

Append-only. Each entry uses one of the following formats:

```
## [YYYY-MM-DD] ingest | [Document Title]
2-sentence note on what was added, what changed, and which workstream was applied.

## [YYYY-MM-DD] lint | Wiki structural audit
2-sentence note on what was checked, what was fixed, and any issues flagged for human review.

## [YYYY-MM-DD] schema | [Change description]
2-sentence note on what structural change was made to the wiki schema or AGENTS.md.
```

---

## On Every Lint, You Will

Run these checks against `wiki/` in order. For each check, fix what is safe to
fix automatically, and surface a numbered list of issues that need human review.
Do not start ingesting new material during a lint pass.

1. **Filenames.** Every `.md` under `wiki/sources/`, `wiki/entities/`, and
   `wiki/concepts/` must be lowercase-hyphenated (e.g. `market-entry-strategy.md`).
   When renaming, also rewrite every `[[wikilink]]` that points at the old name.

2. **Frontmatter.** Every page in those three directories must have YAML
   frontmatter with the keys required by its page type (see Page Format above).
   Missing or empty required keys → fill from the page body if obviously
   inferable, otherwise flag. Source pages missing `workstream:` → default to
   `general`. Source pages missing `bucket:` → attempt auto-detection from page
   body; if still ambiguous, set `bucket: unknown` and flag for human review.
   Entity/concept pages missing `workstreams:` → default to `[general]`.

3. **Wikilinks.** Every `[[Target]]` must resolve to a page in `wiki/sources/`,
   `wiki/entities/`, or `wiki/concepts/`. Match case-insensitively and tolerate
   hyphen vs. space. Report any that do not resolve — do not auto-create
   target pages, since that is an ingest operation.

4. **Orphans.** Any page in `wiki/entities/` or `wiki/concepts/` that is linked
   from zero source pages is an orphan. Report — do not delete (per the
   "never delete pages" rule).

5. **Index coverage.** Every non-deprecated page must appear in the matching
   table in `wiki/index.md`. Add missing rows. For pages with `deprecated: true`
   in frontmatter, append `(deprecated)` to their description rather than
   removing the row. Ensure the Sources table includes both the Bucket and
   Workstream columns.

6. **Markdown hygiene.** Run `markdownlint-cli2 'wiki/**/*.md'`. Install with
   `npm install -g markdownlint-cli2` if missing. Use
   `wiki/.markdownlint-cli2.jsonc` if it exists; otherwise apply defaults with
   `MD013` (line length) and `MD041` (first line must be h1) disabled — wiki
   pages start with frontmatter, not headings.

7. **Log.** Append a lint entry to `wiki/log.md`.
