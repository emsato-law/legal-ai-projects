# Legal AI Projects

Public-facing architecture and design notes for legal AI systems built for real-world practice across ASEAN jurisdictions.

This public repository explains the architecture, design decisions, and system flows behind a portfolio of legal AI projects whose implementation lives in private repositories. It is not a source-code mirror. Its purpose is to document what each system is for, how it is structured, and why it was designed that way.

---

## Table of Contents

- [Portfolio at a Glance](#portfolio-at-a-glance)
- [Projects](#projects)
  - [Translation Pipeline](#translation-pipeline)
  - [Legal Knowledge Base](#legal-knowledge-base)
  - [Legal Wiki](#legal-wiki)
  - [Automated Research Pipelines](#automated-research-pipelines)
  - [Fee Proposal Generator](#fee-proposal-generator)
  - [DOCX Styler](#docx-styler)
  - [SHA-SG](#sha-sg)
- [Design Philosophy](#design-philosophy)
- [About](#about)

---

## Portfolio at a Glance

| Metric | Detail |
|---|---|
| **Combined codebase** | 140,000+ lines (Python, JavaScript, HTML/CSS, Markdown) |
| **Total commits** | 1,900+ across verified active project repositories |
| **Tracked project files** | 900+ files across verified active project repositories |
| **First commit** | 2023 |
| **Stack** | Python · JavaScript · Direct LLM API integration (Gemini, Claude, GPT, MiniMax) · Custom prompt orchestration · Scheduled automations · Full deployment pipelines |

The portfolio’s core knowledge flow is:

`Translation Pipeline -> Legal Knowledge Base -> Legal Wiki`

---

## Projects

### [Translation Pipeline](./translation-pipeline/)
**A complete AI-assisted system built specifically for legal and regulatory document translation** · *In active use*

- **Multi-model routing** — separate models for OCR correction, table reconstruction, and translation, each with task-appropriate temperature tuning
- **Language-aware prompt architecture** — specialized prompts where linguistically necessary (Thai, Bengali), generic prompts where not
- **Anti-runaway chunking** — production-hardened protections against LLM content expansion and infinite recursion in long documents
- **Hybrid extraction routing** — born-digital PDFs go through local native extraction with page-level layout classification; scanned documents and raster images route to a switchable cloud OCR lane (currently, Google Document AI or Datalab)
- **Cloud job architecture** — web control plane backed by queued long-running workers, durable job state, and retained per-step artifacts
- **Partner and partial-workflow APIs** — server-to-server REST access with bearer-token auth, webhook delivery, idempotency keys, cancellation, purge, and step-specific artifact downloads
- **Review-aware output** — warnings, review targets, and structured findings identify where human inspection is needed, rather than silently passing risky output
- **Step 1 to Step 2 diff viewer** — Advanced Mode compares raw conversion output against corrected/normalized Markdown before translation
- **Inputs:** PDF, DOCX, images, public webpages
- **Outputs:** Markdown, DOCX, HTML, PDF
- **System role:** the first stage in the `Translation Pipeline -> Legal Knowledge Base -> Legal Wiki` chain
- **Downstream integration:** outputs designed to feed directly into the [Legal Knowledge Base](./legal-knowledge-base/) as ingestion-ready source material, with the Legal Wiki consuming source-backed citations downstream

<div align="center">
  <img src="./translation-pipeline/screenshots/splash.png" alt="Translation Pipeline — Home" width="600">
  <br><em>Home — Quick Translation and Advanced Mode entry points</em>
  <br><sub>As of 18 March 2026</sub>
</div>

<p>&nbsp;</p>

<div align="center">
  <img src="./translation-pipeline/screenshots/advanced-mode.png" alt="Translation Pipeline — Advanced Mode" width="600">
  <br><em>Advanced Mode — Step-by-step pipeline control with per-stage review</em>
  <br><sub>As of 18 March 2026</sub>
</div>

<p>&nbsp;</p>

<div align="center">
  <img src="./translation-pipeline/screenshots/task-dashboard.png" alt="Translation Pipeline — Task Dashboard" width="600">
  <br><em>Task Dashboard — Production usage tracking across translation jobs</em>
  <br><sub>As of 18 March 2026</sub>
</div>

<p>&nbsp;</p>

```
Pipeline Steps                                        🤖 = AI-powered step
──────────────────────────────────────────────────

Step 1 ─ Convert to Markdown
           ├── DOCX and born-digital PDF →
           │     ├── Local extraction + layout classification
           │     └── Sensitive information anonymization (coming soon) 🤖
           ├── Scanned documents and images →
           │     ├── Sensitive information anonymization (coming soon) 🤖
           │     └── Cloud OCR (switchable: Google Document AI / Datalab) 🤖
           └── Public webpage URL → HTML extraction + Markdown conversion

Step 2 ─ Correct and Normalize
           ├── OCR correction, language-aware 🤖
           ├── Table reconstruction 🤖
           ├── Numbering validation, mixed numeral systems 🤖
           ├── Structural analysis → review findings 🤖
           ├── Text cleanup 🤖
           └── Step 1 ↔ Step 2 cleanup diff viewer

Step 3 ─ Translate
           ├── Language-pair-specific prompt templates 🤖
           ├── Critical-term preservation
           ├── Deterministic substitutions
           ├── Post-translation review checks 🤖
           └── Re-insert sensitive information (coming soon)

Step 4 ─ Export
           ├── Markdown  ─┐
           ├── DOCX       │→ with review signals carried forward
           ├── HTML       │
           └── PDF       ─┘

Partner API ─ Server-to-server integration
           ├── Bearer-token auth with per-partner limits
           ├── Job lifecycle: create, poll, cancel, purge
           ├── Webhook delivery with HMAC signatures
           └── Idempotency keys for safe retries
```

→ [Architecture & Design](./translation-pipeline/README.md)

---

### [Legal Knowledge Base](./legal-knowledge-base/)
**Files-first canonical legal knowledge layer** · *In active development*

The Legal Knowledge Base (LKB) is the authoritative legal source layer for the system: ingestion, versioned Markdown, provenance, validation, audit trail, SQLite-backed search, and reviewed downstream Legal Wiki handoff.

- **System role:** the canonical middle layer in `Translation Pipeline -> Legal Knowledge Base -> Legal Wiki`
- **Designed upstream source:** the [Translation Pipeline](./translation-pipeline/), whose structure-corrected translated output is intended to flow into Legal Knowledge Base ingestion
- **Current status:** ingestion, validation, UI-assisted review, canonical storage, search indexing, and source-backed downstream drafting are operational
- **Post-ingest workflow:** primary law, notes, references, and templates can move from canonical ingest into a staged Legal Wiki review without making the wiki authoritative
- **Downstream role:** the Legal Knowledge Base provides the source-backed citations that the Legal Wiki uses for synthesis pages

<div align="center">
  <img src="./legal-knowledge-base/screenshots/splash.png" alt="Legal Knowledge Base — AI-Assisted KB Ingest" width="600">
  <br><em>AI-Assisted KB Ingest — single-file and primary-law family intake</em>
  <br><sub>As of 26 March 2026</sub>
</div>

<p>&nbsp;</p>

<div align="center">
  <img src="./legal-knowledge-base/screenshots/suggested_meta.png" alt="Legal Knowledge Base — Review Draft" width="600">
  <br><em>Review Draft — human review interface with AI-suggested metadata refinements</em>
  <br><sub>As of 26 March 2026</sub>
</div>

<p>&nbsp;</p>

→ [Architecture & Design](./legal-knowledge-base/README.md)

---

### [Legal Wiki](./legal-wiki/)
**Synthesis-only legal wiki layer** · *In active development*

The Legal Wiki is the living synthesis layer for the legal knowledge system. It turns source-backed Legal Knowledge Base materials into topics, issues, concepts, process pages, entities, and question pages without replacing the underlying authority.

- **System role:** the synthesis layer in `Translation Pipeline -> Legal Knowledge Base -> Legal Wiki`
- **Authority boundary:** the Legal Wiki is not the canonical legal source of truth; controlling legal text, provenance, version history, and authoritative citations live in the Legal Knowledge Base
- **Source grounding:** pages remain explicitly grounded in Legal Knowledge Base citations and should not duplicate full legal text except for short excerpts when necessary
- **Page model:** exactly six page types — topics, issues, concepts, process pages, entities, and questions
- **Authoring flow:** page helpers support blank page creation, source-grounded draft generation, and reviewed post-ingest proposals from the Legal Knowledge Base
- **Browsing model:** plain Markdown designed to work as an Obsidian vault with stable page IDs, backlinks, and graph navigation

→ [Architecture & Design](./legal-wiki/README.md)

---

### [Automated Research Pipelines](./automated-research-pipelines/)
**Scheduled AI-driven research, summarization, and distribution** · *In active use*

A set of autonomous pipelines running on scheduled agent sessions. Each run performs targeted web research, produces structured summaries, and updates dashboard-backed research stores. Current use cases include daily legal and AI briefings, general news monitoring, and tracking company developments, funding activity, and market movements across the AI legal sector.

- **Research modules:** Thai law, AI updates, general news, and AI legal-sector company tracking
- **Dashboard output:** historical item stores, source-backed summaries, tag filtering, date-range views, and company profile pages
- **Feedback loop:** local article feedback is read by the next scheduled run and folded into later tagging, relevance, and prioritization decisions
- **Legacy migration:** older Discord-first dashboard processes are now process references, not runtime dependencies

→ [Architecture & Design](./automated-research-pipelines/README.md)

---

### [Fee Proposal Generator](./fee-proposal-generator/)
**Automated fee proposal drafting for legal engagements** · *Retired — replaced by firm-wide legal AI platform*

Generates structured fee proposals from intake parameters — scope, personnel, billing/deposit requirements. Reduces a 2–3 hour manual drafting process to minutes while maintaining firm-specific formatting and compliance with engagement standards.

→ [Architecture & Design](./fee-proposal-generator/README.md)

---

### [DOCX Styler](./docx-styler/)
**AI-driven paragraph styling for Word documents** · *Retired — replaced by firm-wide legal AI platform*

Applies consistent paragraph-level styling to legal Word documents using AI classification. Addresses the endemic problem of inconsistent formatting in documents that pass through multiple hands — associates, partners, clients, opposing counsel — before final production.

→ [Architecture & Design](./docx-styler/README.md)

---

### [SHA-SG](./sha-sg/)
**Singapore venture capital templates under version control** · *In use*

Places standard Singapore law venture capital template agreements (shareholders' agreement, subscription agreement, constitution) into structured version control. Tracks clause-level changes across template revisions and enables diffing between template versions.

→ [Architecture & Design](./sha-sg/README.md)

---

## Design Philosophy

These systems share a common set of architectural principles:

- **Direct API integration over frameworks.** No LangChain, no LlamaIndex. Every LLM call is a deliberate, auditable API call with custom prompt management — essential when outputs carry legal consequences.

- **Model routing by task complexity.** Lightweight models handle routine extraction and classification. Capable models handle reasoning, drafting, and ambiguous interpretation. Cost and speed stay proportional to difficulty.

- **Canonical first, synthesis second.** The Legal Knowledge Base holds controlling legal text and citations. The Legal Wiki organizes, compares, and explains that material without replacing it.

- **Legal-native data models.** Clause hierarchies, numbered sections, mixed numeral systems, and bilingual instrument pairs are modeled as first-class structures, not flattened into plain text.

- **Review-aware over silently confident.** Output that looks correct but has hidden defects is a serious risk for legal work. These systems surface uncertainty rather than hide it — structured review findings and explicit warnings are first-class features.

---

## About

- **Background:** U.S.-qualified technology lawyer at DFDL Legal & Tax
- **AI leadership roles:** AI Strategic Initiative committee member, regional lead for Knowledge Managers, and Legal AI Workflows Champion
- **Builder:** designer and developer of the AI systems documented here — 140,000+ lines of production code and documentation across legal translation, knowledge management, document automation, and workflow orchestration
- **Region:** based in ASEAN, working across Thai, Singaporean, and regional legal frameworks

For more information: [emsato.com](https://emsato.com) or [LinkedIn](https://www.linkedin.com/in/emsato/)

---

*Last updated: 10 May 2026*
