# Legal Wiki

**Synthesis-only legal wiki layer built on top of Legal Knowledge Base citations**

The Legal Wiki (LW) is the living synthesis layer for the legal knowledge system.

It is not the canonical legal source of truth. Controlling legal text, provenance, version history, and authoritative citations live in the Legal Knowledge Base (LKB).

The Legal Wiki is designed to help lawyers move from source retrieval to structured understanding without blurring the boundary between controlling legal text and explanatory synthesis.

**Status:** Source-grounded page generation, page helpers, Obsidian-first navigation, page contract checks, and human-reviewed synthesis workflows are in active development.

The Legal Wiki organizes and explains Legal Knowledge Base materials, but it does not replace controlling authority.

---

## Problem

Canonical legal text is necessary, but it is not enough for day-to-day legal work.

Lawyers often need more than retrieval:

1. A topic page that groups related instruments, guidance, and issues.
2. A process page that explains procedural sequence, approvals, dependencies, and practical timing.
3. A synthesis page that compares authorities, highlights unresolved questions, and points to the controlling source.

General-purpose wikis usually fail this job in two ways:

1. They duplicate source text without clear authority boundaries.
2. They produce summaries without a durable citation trail back to the underlying legal materials.

The Legal Wiki is meant to solve that gap. It sits on top of the Legal Knowledge Base and produces useful synthesis while keeping every page grounded in source-backed citations.

---

## Core Boundary

- The Legal Knowledge Base is controlling authority for legal text.
- The Legal Wiki is synthesis-only.
- The Legal Wiki may summarize, compare, and interpret Legal Knowledge Base materials.
- The Legal Wiki must not duplicate full legal text except for short excerpts when necessary.
- If the Legal Wiki conflicts with the Legal Knowledge Base, the Legal Wiki is wrong until reviewed.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│        LEGAL KNOWLEDGE BASE (AUTHORITATIVE)         │
│                                                     │
│  Canonical legal text · Provenance · Audit trail    │
│  Structured metadata · Search index · Citations     │
└──────────────────────┬──────────────────────────────┘
                       │  cited fragments + metadata
                       ▼
┌─────────────────────────────────────────────────────┐
│          LEGAL WIKI DRAFTING AND SYNTHESIS          │
│                                                     │
│  Blank page helpers                                 │
│  Source-grounded draft generation                   │
│  Topic and issue framing                            │
│  Cross-source comparison                            │
│  Process and entity assembly                        │
└──────────────────────┬──────────────────────────────┘
                       │  human review + editorial gate
                       ▼
┌─────────────────────────────────────────────────────┐
│               LEGAL WIKI PAGES (LW)                 │
│                                                     │
│  Topics · Issues · Concepts · Processes             │
│  Entities · Questions                               │
│  Explanation with citations, not replacement text   │
└─────────────────────────────────────────────────────┘
```

The operational relationship is:

`Translation Pipeline -> Legal Knowledge Base -> Legal Wiki`

The Translation Pipeline produces structured source material. The Legal Knowledge Base stores and validates the canonical legal layer. The Legal Wiki synthesizes that material into lawyer-friendly pages that stay anchored to the Legal Knowledge Base.

---

## Page Model

The Legal Wiki is designed around recurring page shapes rather than a single generic note type:

- **Topic pages** — broad subject-area overviews that gather the relevant authorities, sub-issues, and practical entry points.
- **Issue pages** — focused pages on a specific legal question, ambiguity, or recurring practitioner problem.
- **Concept pages** — definitional pages for terms, doctrines, and framework concepts that recur across instruments.
- **Process pages** — procedural views that track sequence, filings, approvals, dependencies, and practical timing.
- **Entity pages** — pages centered on regulators, ministries, agencies, courts, or other legally relevant actors.
- **Question pages** — direct-answer pages for recurring research prompts, always tied back to cited source material.

These page types share a common rule: synthesis is allowed, but authority must remain legible.

Topic pages are intended to behave as regime hubs or index pages, not one-law summaries. A topic should point to narrower concept, issue, question, process, and entity pages where those relationships are real.

---

## How It Works

### Source-Grounded Drafting

The Legal Wiki supports three main authoring paths:

1. **Blank page creation** for new topics or emerging issues where the structure needs to be designed by the editor.
2. **Source-grounded draft generation** where the system starts from Legal Knowledge Base-backed materials and assembles a first-pass synthesis with citations, source-backed statements, and open questions.
3. **Post-ingest proposal review** where a completed Legal Knowledge Base ingest can propose Legal Wiki pages or updates, but writes nothing until the reviewer approves a draft.

In all three cases, the Legal Knowledge Base provides the source backbone. The wiki page is the explanation layer, not the source of truth.

### Obsidian-First Navigation

The Legal Wiki stays plain Markdown, but it is designed to be browsed as an Obsidian vault:

- filenames match stable page IDs
- internal note links use Obsidian wikilinks
- topic pages act as hub notes for synthesis clusters
- selected vault defaults support graph, backlink, outgoing-link, properties, and search workflows
- Markdown conventions remain the source of truth, not editor workspace state

This gives the synthesis layer a graph-like browsing surface while keeping every page diffable, reviewable, and version-controlled.

### Reviewed Proposal Flow

After successful Legal Knowledge Base ingest, the reviewer can choose whether to begin Legal Wiki review. The proposal flow is staged:

- topic overview
- placement review against existing wiki pages
- draft review
- explicit approve or reject

The proposal engine can suggest any of the six page types and can show update proposals when an existing page is the right target. Generated drafts remain drafts until reviewed.

### Human Review Boundary

Human review remains required before a page should be treated as publishable work product.

The review pass checks:

- whether the cited authorities actually support the synthesized statement
- whether the page is overstating certainty where the underlying sources are ambiguous
- whether short excerpts are used only where needed and not as a substitute for structured explanation
- whether the page distinguishes controlling authority from commentary, practice guidance, or inference

### Citation Discipline

The Legal Wiki is designed to cite back into the Legal Knowledge Base rather than duplicate full instruments.

That keeps the synthesis layer compact, preserves provenance, and makes it possible to update or correct explanatory pages when the underlying source material changes.

---

## Design Decisions

**Why separate the Legal Wiki from the Legal Knowledge Base?**
Because canonical legal text and explanatory synthesis serve different functions. The Legal Knowledge Base stores source material, provenance, and version history. The Legal Wiki compares, organizes, and explains that material. Keeping the layers separate makes it easier to preserve authority boundaries.

**Why require Legal Knowledge Base-backed citations?**
Because legal synthesis without a durable source trail becomes hard to trust and hard to maintain. The citation link back to the Legal Knowledge Base is what keeps the wiki grounded.

**Why support blank-page, source-grounded, and post-ingest workflows?**
Because some pages begin with a clearly defined source set, while others start from an emerging research question that still needs structure. Post-ingest proposals cover a third case: the source has just become canonical in the Legal Knowledge Base and should be reviewed for possible synthesis. The Legal Wiki needs to handle these modes without collapsing them into the same workflow.

**Why use an Obsidian-first browsing model?**
Because synthesis pages are more useful when a lawyer can move through related topics, concepts, entities, and issues quickly. Obsidian-style links provide that navigation without requiring the wiki to stop being plain Markdown.

**Why keep human review in the loop?**
Because synthesis is where overstatement risk becomes acute. Even when the source retrieval is correct, the summary, comparison, or issue framing can still be wrong or too confident. The review boundary is a product requirement, not an implementation detail.

---

*→ Back to [Project Index](../README.md)*
