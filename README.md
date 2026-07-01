# LLM-Wiki for CSPs: A Self-Maintaining Knowledge Base for AI-Assisted Consulting

> Turn a folder of client documents into a structured, AI-queryable knowledge
> base that builds and updates itself. No infrastructure, no technical
> background required.

---

## Table of Contents

1. [What Is an LLM-Wiki?](#1-what-is-an-llm-wiki)
2. [What Is It Useful For?](#2-what-is-it-useful-for)
3. [How to Set It Up](#3-how-to-set-it-up)
4. [How to Use It](#4-how-to-use-it)
5. [Customizing AGENTS.md for Your Engagement](#5-customizing-agentsmd-for-your-engagement)
6. [Advanced: Why LLM-Wiki vs. Classic RAG, and Multi-Wiki Federation](#6-advanced-why-llm-wiki-vs-classic-rag-and-multi-wiki-federation)

---

## 1. What Is an LLM-Wiki?

### The problem it solves

Imagine you are three months into an engagement. You have 180 documents:
the client's annual reports, board strategy presentations, financial
filings, prior analyses your team has produced, and a stack of industry
research. You're asked to pull together a two-page briefing on the
client's core strategic tensions before a meeting with the key sponsor 
the next morning.

Without a wiki, you re-read documents you may not remember, search your
email for the right deck, and are constantly hunting for the correct data
source for your content, hoping you havn't missed something important. 

**With an LLM-Wiki, you open a chat and type:**

> *"Summarize the client's three biggest strategic tensions. Cite your sources."*

And the agent responds with something like:

> **Strategic tension 1: Cost structure mismatch.** The client's operating
> cost base has grown 18% over two years while revenue has grown only 7%,
> creating sustained margin compression. This pattern is documented across
> three annual reports (→ `sources/annual-report-2022`,
> `sources/annual-report-2023`, `sources/annual-report-2024`) and flagged
> by two external analysts (→ `sources/analyst-note-jan-2024`).
>
> **Strategic tension 2: Geographic concentration.** Approximately 70% of
> revenue comes from two domestic markets, but the client has publicly
> committed to international expansion by 2026. The tension between
> near-term margin pressure and long-term growth investment is documented
> across four board strategy presentations
> (→ `sources/board-strategy-q1` through `q4`).
>
> *(and so on...)*

Every claim is traceable to a specific ingested document. 
### Why not just dump your documents into an AI?

You can already paste documents into ChatGPT or Lilli. So why build a
wiki at all?

The difference is **accumulation**. When you upload files to a standard
AI tool and ask a question, the model is rediscovering knowledge from
scratch: retrieving relevant chunks and piecing together an answer on the
fly. Ask a subtle question that requires synthesizing five documents and
the model has to find and piece together the relevant fragments every
single time. Add a new document and the model has no memory of the
connections it found last session.

An LLM-Wiki works differently. When you add a new source, the agent does
not just index it for later retrieval. It reads it, extracts the key
information, and **integrates it into the existing wiki**, updating entity
pages, revising topic summaries, and noting where new data contradicts old
claims. The knowledge is compiled once and kept current, not re-derived on
every query.

The result is a **persistent, compounding database**.

### How it works

An LLM-Wiki has three layers:

1. **Raw sources.** Your original documents, untouched. PDFs, PowerPoints,
   Word files, transcripts, spreadsheet exports. The agent reads from them
   but never modifies them.

2. **The wiki.** A folder of agent-generated Markdown files. A **source
   page** for each document (summary, key data points, and links to every
   entity and concept it mentions), an **entity page** for each named
   company, person, or product (facts aggregated across every source that
   mentions them), and a **concept page** for each key theme or framework.
   The agent writes and maintains all of this; you read it. Periodically
   you can also ask the agent to run a **lint pass**: an automated audit
   that checks for broken links, stale claims, missing metadata, and
   orphaned pages, and fixes most issues automatically.

3. **The schema (`AGENTS.md`).** A plain-text instruction file that tells
   the agent how the wiki is structured, what conventions to follow, and
   what to do when ingesting a source, answering a query, or running a
   lint pass. This is the only configuration file you need. Drop it into
   any folder and the agent becomes a disciplined wiki maintainer rather
   than a generic chatbot.

Your job is to curate sources, direct the analysis, and ask the right
questions. The agent handles everything else: summarizing,
cross-referencing, filing, and flagging contradictions.

The wiki lives entirely in plain text files, with no database or server
required. It opens natively in Obsidian (downloadable from GHD), which
renders every cross-reference as a clickable link and can provide a graph
view of the entire knowledge base.

### Core vocabulary

| Term             | Meaning                                                                                                                                         |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **Ingest**       | Reading a new document and writing wiki pages from it                                                                                           |
| **Source page**  | A summary page for one ingested document (`wiki/sources/`)                                                                                      |
| **Entity page**  | A persistent page for a named company, person, product, or org (`wiki/entities/`)                                                               |
| **Concept page** | A persistent page for a theme, framework, or market dynamic (`wiki/concepts/`)                                                                  |
| **Bucket**       | Who authored the document (e.g., `client-internal`, `external-research`)                                                                        |
| **Workstream**   | What consulting task the document feeds (e.g., `competitive-analysis`, `market-sizing`)                                                         |
| **Index**        | A single file (`wiki/index.md`) listing every page in the wiki with a one-line description; updated automatically on every ingest               |
| **Log**          | An append-only file (`wiki/log.md`) recording every ingest, query, and audit with a timestamp; useful for tracking what has been added and when |
| **Lint pass**    | An automated structural audit of the wiki that fixes common issues                                                                              |

### What the pages look like

Three page types work together. Notice how the cross-links (`[[...]]`)
connect them: the source page links to the entity and concept, the entity
lists its sources, and the concept ties back to both. In Obsidian, every
`[[link]]` renders as a clickable connection and appears in the graph view.

<table>
<tr>
<th>Source page<br><code>wiki/sources/annual-report-2024.md</code></th>
<th>Entity page<br><code>wiki/entities/acme-industrial.md</code></th>
<th>Concept page<br><code>wiki/concepts/margin-compression.md</code></th>
</tr>
<tr>
<td valign="top"><pre>
---
title: Annual Report 2024
source_file: raw/annual-report-2024.pdf
date_ingested: 2026-05-10
type: report
bucket: client-internal
workstream: general
---

# Annual Report 2024

## Summary
FY2024 filing for Acme Industrial.
Operating margins declined for the
third consecutive year as cost growth
(18%) outpaced revenue growth (7%).
Management flagged international
expansion as a strategic priority
but noted near-term budget constraints.

## Key Data Points
- Revenue: $4.2B (+7% YoY)
- Operating margin: 11.8%
  (down from 14.2% in 2022)
- North America: 71% of revenue

## Entities Referenced
- [[acme-industrial]] — filing company
- [[north-america-segment]] — primary unit

## Concepts Referenced
- [[margin-compression]] — core theme
- [[geographic-concentration]] — risk flag

## Contradictions / Open Questions
CFO states cost initiatives are "on
track" but margins fell again. Conflicts
with [[investor-day-2024]] guidance
of stabilization by H2 2024.
</pre></td>
<td valign="top"><pre>
---
title: Acme Industrial
type: company
sources:
  - annual-report-2024
  - investor-day-2024
  - analyst-note-jan-2024
workstreams: [general, client-assessment]
---

# Acme Industrial

## Overview
Large industrial manufacturer and
primary engagement client. Operations
in North America and Western Europe.

## Key Facts
- FY2024 revenue: $4.2B
- Operating margin: 11.8% (2024)
  vs 14.2% (2022)
- Employees: ~18,000

## Source Appearances
- [[annual-report-2024]] — annual filing
- [[investor-day-2024]] — strategy deck
- [[analyst-note-jan-2024]] — risk review

## Related
- [[margin-compression]] — key challenge
- [[geographic-concentration]] — risk
</pre></td>
<td valign="top"><pre>
---
title: Margin Compression
type: theme
sources:
  - annual-report-2024
  - analyst-note-jan-2024
workstreams: [general, client-assessment]
---

# Margin Compression

## Overview
Sustained operating margin decline
driven by cost growth consistently
outpacing revenue growth, observed
from 2022 through 2024.

## Key Facts
- Client margin: 11.8% (2024)
  vs 14.2% (2022)
- Industry peer avg: ~15.5%
  per [[analyst-note-jan-2024]]

## Source Appearances
- [[annual-report-2024]] — trend data
- [[analyst-note-jan-2024]] — benchmarks

## Related
- [[acme-industrial]] — affected entity
- [[geographic-concentration]] — risk
</pre></td>
</tr>
</table>

---

## 2. What Is It Useful For?

### The meso-scale sweet spot

Most consulting engagements generate a corpus in the range of **50 to 500
documents**. This range is the hardest to work with:

- **Too large** to keep in your head or re-read before every deliverable
- **Too small** to justify building a dedicated search pipeline
- **Too heterogeneous** (PDFs, PowerPoints, Word docs, spreadsheets)
  for simple keyword search

Because every document is summarized and cross-referenced at ingest time,
queries can be answered by reading a handful of well-organized wiki pages
rather than re-scanning the full raw corpus, and the answers come back
with citations you can verify.

### What the outputs look like

The agent is not limited to what the wiki contains: it can augment its
answers with its own reasoning or pull in live external research when
useful, clearly labeling what came from the wiki versus outside sources.

| What you ask                                                                                  | What you get                                                                                                                      |
| --------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| *"What are the client's top strategic priorities?"*                                           | A bulleted synthesis with citations to source documents, with any contradictions between sources flagged                          |
| *"Draft an executive summary of our current hypothesis on the client's competitive position"* | A narrative draft grounded in your ingested materials, with the model's own reasoning filling gaps where the documents are silent |
| *"What does the external research say about best-practice operating margins in this sector?"* | A scoped answer drawn only from external-research sources, with a note on how the client's situation compares based on the wiki   |
| *"Are there any conflicting claims about [Topic X] across our materials?"*                    | A structured list of contradictions, each citing the two source documents that disagree                                           |
| *"What do we know about [Company Y] from everything we have ingested?"*                       | The full entity page for Company Y, with facts aggregated across every source that mentions them                                  |

Once the wiki is built, you can run any number of queries across any
combination of document types without re-reading anything. New team
members can get up to speed by querying the wiki directly.

### Self-maintaining as the engagement grows

The wiki is designed to grow alongside your work:

- **Incremental ingestion.** Drop a new document in and run one command.
  The agent adds the new source page, updates every entity and concept
  page it touches, and logs the change.
- **Contradiction detection.** When a newly ingested document disagrees
  with something already in the wiki, the agent flags the conflict on
  both the new source page and the relevant entity/concept page, so you
  are never silently working with outdated information.
- **Lint passes.** A single audit command checks the entire wiki for
  broken links, missing metadata, orphaned pages, and formatting issues,
  and fixes most of them automatically.

### Workstream isolation

A single engagement may have several parallel work tracks (a market
sizing effort, a competitive analysis, an internal assessment) that
should not contaminate each other's deliverables. The wiki supports this
through **workstream tags** on every page.

When you query for a specific deliverable, the agent draws only from
sources tagged for that workstream (plus anything tagged `general`). This
means that sensitive client operational data ingested for an internal
assessment will never leak into an externally shared market report, even
if both sets of documents live in the same wiki.

### Example use cases

#### Example A: Understanding a client before a strategy kick-off

A consulting team is preparing to launch a cost transformation engagement
for a large industrial company. Before the first client meeting, a team
member ingests thirty documents: the last three annual reports, several
investor day presentations, a competitor benchmarking study, and prior
internal analyses from a related engagement.

They ask:

> *"What are the client's most significant operational inefficiencies
> based on what they have disclosed publicly, and which business units
> appear most exposed to margin pressure?"*

The agent returns a grounded synthesis with citations drawn from those
documents.

#### Example B: Separating external benchmarks from client-specific findings

A team is assessing the digital maturity of a regional bank. They ingest
the client's internal strategy documents and self-assessment materials
tagged `workstream: client-assessment`, and separately ingest industry
research, regulator guidance, and peer bank disclosures tagged
`workstream: external-benchmarks`.

They ask:

> *"Using only the external benchmark materials, what digital capabilities
> do leading regional banks have in common, and how are those capabilities
> typically organized?"*

The agent scopes strictly to `external-benchmarks` and returns a clean
synthesis. The team can then manually compare that picture against the
client-assessment workstream findings, with no risk of the client's
proprietary data appearing in an external-facing deliverable.

#### Example C: Reusing knowledge from prior engagements for a new proposal

A team is preparing materials for a new business opportunity in a sector
where the firm has done significant prior work. They have a wiki from
several past engagements containing methodology write-ups, anonymized
case study summaries, and prior analytical frameworks, all ingested as
`general`.

They ask:

> *"Based on our prior work in this sector, what are the two or three
> structural factors that most consistently drive successful cost
> transformation programs? Draft a one-page narrative I can use as the
> backbone of a pitch document."*

The agent draws on the accumulated prior-work wiki and produces a draft
grounded in the firm's actual track record rather than generic
best-practice language. Because bucket tags distinguish client-sourced
materials from firm-authored methodology, the agent never accidentally
attributes a past client's approach to the firm's own IP.

---

## 3. How to Set It Up

### Prerequisites

You need:

1. **Cursor** — available from GHD.
2. **Obsidian** *(optional but recommended)* — a Markdown editor with a
   graph view that makes the wiki pleasant to browse, also available
   from GHD.

No Python environment, no API keys beyond what Cursor already uses, and
no database are required.

### Step 1: Create your engagement folder and set up AGENTS.md

Create a new folder on your computer for this engagement (you can name
it anything, e.g. `client-name-wiki`). Download `AGENTS_TEMPLATE.md`
from this repository, drag it into that folder, and rename it to
`AGENTS.md`. That file is the only configuration the agent needs.

Before ingesting anything, make these quick edits to `AGENTS.md`:

1. **Update the Buckets table** near the top. It comes with two generic
   rows (`client-internal` and `external-research`). Rename or add rows
   to match the document types in your corpus (for example, adding a row
   for a specific research firm or regulator). See Section 5 of this
   README for ideas.
2. **(Optional) Document your workstreams** in the Workstreams section.
   Workstreams are also created automatically from your
   ingest commands, but this step can give your agent more context.
3. Everything else can be left exactly as-is.

You can also skip all customization for now and just ask Cursor to update the
file later once you have started ingesting.
### Step 2: Open the folder in Cursor and build the structure

Open your engagement folder in Cursor using **File > Open Folder** and
selecting the folder you just created. Then open a new chat with the
Cursor agent and paste this message:

```
Create my wiki folder structure. Make the following folders: raw, wiki,
wiki/sources, wiki/entities, wiki/concepts. Then create wiki/index.md
containing only the heading "# Wiki Index" and wiki/log.md containing
only the heading "# Wiki Log".
```

The agent will create everything.

### Step 3: Add the AGENTS.md Cursor rule

This step tells Cursor to always read `AGENTS.md` before doing anything,
so it never forgets the wiki conventions mid-engagement. In the same chat
(or a new one), paste:

```
Create the file .cursor/rules/read-agents-first.mdc with exactly this
content:

# Workspace bootstrap

Always read AGENTS.md before taking any action in this workspace.
```

### Step 4 (optional): Open in Obsidian

If you have Obsidian installed, open your engagement folder as a vault
using **File > Open Folder as Vault**. Every `[[link]]` the agent writes
will immediately render as a clickable connection, and the graph view
will show you a visual map of how all the wiki pages relate to each
other.

That is the entire setup. Your wiki is ready.

---

## 4. How to Use It

All interaction happens through plain-language chat in a Cursor Agent
session. Open the engagement folder in Cursor, start a new chat, and
type.

### Ingesting documents

**The recommended way: drag files into the chat.**

Open a new Cursor chat and drag one or more documents directly into the
chat window from anywhere on your computer, including OneDrive,
SharePoint, or any local folder. Then type:

> `Ingest these`

Or, to assign them to a workstream:

> `Ingest these for Workstream competitive-analysis`

The agent will automatically copy the files into the `raw` folder inside
your wiki, then read and process each one. You never need to move files
manually or know their exact names.

**Alternative: reference files by name.**

If your files are already in the `raw` folder, or if you have a filename
easily at hand, you can skip the drag-and-drop and reference it directly:

> `Ingest raw/filename.pdf`

> `Ingest raw/filename.pdf for Workstream market-sizing`

Either way, the agent will:

- Copy the file into the `raw` folder if it is not already there
- Convert it to readable text (it handles PDF, PowerPoint, and Word)
- Automatically detect the bucket based on the document's authorship
- Write a summary page to the wiki
- Create or update the relevant entity and concept pages
- Update the index and log

**Specifying the bucket manually.**

The agent auto-detects the bucket from the document's title, cover page,
and header. If it cannot determine the authorship confidently it will flag
the document for review. You can always override the auto-detection by
stating the bucket explicitly in your ingest command:

> `Ingest these as client-internal`

> `Ingest these as external-research for Workstream competitive-analysis`

You can combine bucket and workstream in any order. If you have added
custom bucket names to `AGENTS.md` (see Section 5), use those names in
the same way:

> `Ingest these as regulator-guidance for Workstream client-assessment`

### Querying the wiki

Start a new chat session for each query.

You can ask anything the ingested documents can support:

| What you want | What to type |
|---|---|
| A synthesis across all sources | `"Summarize the key themes across all ingested documents"` |
| A workstream-scoped deliverable | `"Using only the external-research workstream, summarize the competitive landscape"` |
| A grounded analytical draft | `"Draft a one-page hypothesis on the client's core strategic challenge. Draw on all available materials."` |
| A fact-check | `"What does the wiki say about [Company X]'s market position? Cite your sources."` |
| A contradiction check | `"Are there any conflicting claims about [Topic Y] across our sources?"` |

### Running a lint pass

After a large batch of ingestions, or periodically as the engagement
grows, run a structural audit:

> `Lint the wiki`

The agent checks for broken wikilinks, missing frontmatter, orphaned
pages, index coverage gaps, and Markdown formatting issues, fixes what it
can automatically, and reports anything that needs human review.

### Tips for non-technical users

- **One chat session, one task.** Start a new chat for each ingest and
  each query. Do not try to ingest and query in the same session.
- **You do not need to know where files are.** The agent finds and reads
  wiki pages itself. You just ask questions in plain English.
- **Your original documents are never modified.** The agent only writes
  to the `wiki` folder. Everything in `raw` stays exactly as you left it.
- **Workstream names are up to you.** Use whatever names reflect your
  engagement structure: `market-sizing`, `client-assessment`,
  `competitive-analysis`, `pitch-prep`, etc. Just be consistent; once a
  workstream name is set it should not change.

---

## 5. Customizing AGENTS.md for Your Engagement

The `AGENTS.md` file is the agent's instruction manual, and it is designed
to be edited. The template ships with sensible defaults, but tailoring it
to your engagement makes the wiki significantly more useful. You can update
it at any time, even mid-engagement, and the agent will pick up the changes
at the start of the next session.

To make an update, open `AGENTS.md` in Cursor or any text editor, make
your changes, and save. You can also ask Cursor directly:

> `Update AGENTS.md to add a new bucket called "regulator-guidance" for documents issued by regulatory agencies.`

Below are the most useful things to customize, and you can always just ask Cursor to make the customization in the `AGENTS.md` file for you. 

### Buckets

The most common customization. Add a row to the Buckets table for each
distinct document source type in your corpus. Being specific here is key: the agent uses bucket tags to enforce scoping at query time, so a
well-labeled corpus produces cleaner, more trustworthy outputs.

Examples:
- A private equity due diligence engagement might add buckets for
  `target-company`, `market-research`, and `legal-review`
- A regulatory strategy engagement might add `regulator-guidance` and
  `industry-association` alongside `client-internal`
- A competitive intelligence project might break `external-research` into
  `competitor-filings`, `analyst-reports`, and `news-media`

### Entity types

The default entity types (`company`, `product`, `person`, `trial`,
`institution`) cover most cases, but you can extend them. If your
engagement involves a specific category that will appear repeatedly, adding
it as a recognized type helps the agent create more consistent pages.

Examples: `fund`, `portfolio-company`, `regulation`, `clinical-trial`,
`technology-platform`, `business-unit`.

### Ingest instructions

You can add standing instructions for how the agent should handle specific
document types. This is useful when your corpus has a recurring format
that has domain-specific data worth capturing consistently.

Examples:
- "For any financial filing, always extract revenue, EBITDA margin, and
  net debt figures verbatim in the Key Data Points section."
- "For meeting notes, always create an entity page for each attendee
  organization if one does not already exist."
- "For analyst reports, always note the publishing firm and date in the
  source page summary."

### Output formatting preferences

You can specify how you want the agent to structure its responses to
certain types of queries. This is particularly useful if you are
generating content that feeds into a standard deliverable format.

Examples:
- "When asked for a synthesis, structure the response as: situation,
  complication, so what."
- "When drafting narrative content, write in third person and avoid
  bullet points unless specifically asked."

### Workstream definitions

Although workstreams are created on the fly from your ingest commands, you
can document the intended workstreams in `AGENTS.md` upfront. This gives
the agent context about what each workstream is for, which improves
scoping decisions when workstream names are ambiguous.

Example addition to `AGENTS.md` (again, just copy this into the chat and tell Cursor to make the change):

```
## Workstream Definitions

| Workstream name | Purpose |
|---|---|
| `client-assessment` | Internal-facing analysis of the client's current state |
| `external-benchmarks` | External research for benchmarking only; no client data |
| `pitch-prep` | Materials for the new business pitch; draw on general and prior-work sources only |
```

---

## 6. Advanced: Why LLM-Wiki vs. Classic RAG, and Multi-Wiki Federation

*This section is for technically curious readers. None of it is required
to use the system.*

### Why not just use retrieval-augmented generation (RAG)?

Classic RAG works by converting documents into vector embeddings and
storing them in a vector database. At query time, the system finds the
most semantically similar chunks and feeds them to the language model as
context. RAG is powerful and well-understood, but it has several
limitations at the meso-scale that an LLM-Wiki sidesteps:

| Dimension | Classic RAG | LLM-Wiki |
|---|---|---|
| **Infrastructure** | Requires embedding model, vector DB, and retrieval pipeline | Plain Markdown files; no infrastructure |
| **Chunk granularity** | Retrieval works at paragraph level; cross-document synthesis requires many retrieval passes | Entity and concept pages aggregate information across all sources at ingest time |
| **Contradiction handling** | Contradicting chunks are both retrieved and silently passed to the model | Contradictions are detected and flagged at ingest time; the model is told about them explicitly |
| **Auditability** | Hard to know which raw chunks informed an answer | Every wiki page carries source citations; reasoning is inspectable |
| **Incremental update** | Re-embedding and re-indexing is required when documents change | Append new source pages, update affected entity/concept pages, log the change |
| **Workstream isolation** | Typically requires separate indexes or metadata filtering | Workstream tags on every page; agent enforces scoping rules in natural language |
| **Setup cost** | Significant for non-technical users | Copy an `AGENTS.md` file |

The core tradeoff is that **synthesis happens at ingest time rather than
query time**. When you ingest a document the agent reads it in full and
produces a cross-referenced summary. When you query later you are reading
those structured pages rather than raw document chunks.

This comes at a cost: ingestion is slower and more expensive than
embedding because the language model reads every document carefully. For
very large corpora (thousands of documents) classic RAG is likely more
practical. For the meso-scale range of a typical consulting engagement
the quality improvement is worth the tradeoff.

### Multi-wiki federation

As engagements grow, or as a firm wants to reuse knowledge across
multiple client relationships, it becomes useful to maintain **multiple
separate LLM-Wikis** that can be queried together without
cross-contaminating each other's proprietary content.

A federated architecture might look like this:

```
firm-knowledge-wiki/        <- internal methodology, frameworks, prior work
    wiki/sources/
    wiki/entities/
    wiki/concepts/

client-a-wiki/              <- everything related to Client A
    wiki/sources/
    wiki/entities/
    wiki/concepts/

client-b-wiki/              <- everything related to Client B
    wiki/sources/
    wiki/entities/
    wiki/concepts/
```

Each wiki has its own `AGENTS.md` with its own bucket and workstream
definitions. Client A's data never touches Client B's wiki.

To query across wikis, an AI agent can be given read access to multiple
wiki folders simultaneously and instructed to:

1. Retrieve relevant pages from each wiki independently.
2. Synthesize a combined answer, labeling each piece of evidence with
   its wiki of origin.
3. Flag any cases where information from one wiki (such as a general
   industry benchmark) appears to conflict with information from another
   (such as a specific client's reported metrics).

Each wiki's isolation is preserved while still enabling a combined answer,
with each piece of evidence labeled by its wiki of origin.

In practice, Cursor's multi-folder workspace feature (or a combined vault
in Obsidian) makes this straightforward. Point the agent at two or more
wiki folders and ask it to draw from all of them while labeling its
sources.
