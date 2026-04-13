# Legal Wiki

**Synthesis-only legal wiki layer built on top of Legal Knowledge Base citations**

The Legal Wiki (LW) is the living synthesis layer for the legal knowledge system.

It is not the canonical legal source of truth. Controlling legal text, provenance, version history, and authoritative citations live in the Legal Knowledge Base (LKB).

The Legal Wiki is designed to help lawyers move from source retrieval to structured understanding without blurring the boundary between controlling legal text and explanatory synthesis.

**Status:** Source-grounded page generation, page helpers, and human-reviewed synthesis workflows are in active development.

The Legal Wiki organizes and explains Legal Knowledge Base materials, but it does not replace controlling authority.

---

## Problem

Canonical legal text is necessary, but it is not enough for day-to-day legal work.

Lawyers often need more than retrieval:

1. A topic page that groups related instruments, guidance, and issues.
2. A timeline that shows how a rule changed over time.
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
│  Timeline and entity assembly                       │
└──────────────────────┬──────────────────────────────┘
                       │  human review + editorial gate
                       ▼
┌─────────────────────────────────────────────────────┐
│               LEGAL WIKI PAGES (LW)                 │
│                                                     │
│  Topics · Issues · Concepts · Timelines             │
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
- **Timeline pages** — change-over-time views that show amendments, effective dates, and supersession chains.
- **Entity pages** — pages centered on regulators, ministries, agencies, courts, or other legally relevant actors.
- **Question pages** — direct-answer pages for recurring research prompts, always tied back to cited source material.

These page types share a common rule: synthesis is allowed, but authority must remain legible.

---

## How It Works

### Source-Grounded Drafting

The Legal Wiki supports two main authoring paths:

1. **Blank page creation** for new topics or emerging issues where the structure needs to be designed by the editor.
2. **Source-grounded draft generation** where the system starts from Legal Knowledge Base-backed materials and assembles a first-pass synthesis with citations, source-backed statements, and open questions.

In both cases, the Legal Knowledge Base provides the source backbone. The wiki page is the explanation layer, not the source of truth.

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

**Why support both blank-page and source-grounded workflows?**
Because some pages begin with a clearly defined source set, while others start from an emerging research question that still needs structure. The Legal Wiki needs to handle both modes without collapsing them into the same workflow.

**Why keep human review in the loop?**
Because synthesis is where overstatement risk becomes acute. Even when the source retrieval is correct, the summary, comparison, or issue framing can still be wrong or too confident. The review boundary is a product requirement, not an implementation detail.

---

*→ Back to [Project Index](../README.md)*
