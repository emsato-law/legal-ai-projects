# Legal AI Projects

Legal AI tools built for real-world practice across ASEAN jurisdictions.

This repository documents the architecture, design decisions, and system flows behind a portfolio of legal AI tools. Source code is maintained in private repositories; what follows is the engineering reasoning behind each system.

---

## Table of Contents

- [Portfolio at a Glance](#portfolio-at-a-glance)
- [Projects](#projects)
  - [Translation Pipeline](#translation-pipeline)
  - [Legal Knowledge Base](#legal-knowledge-base)
  - [Legal Wiki](#legal-wiki)
  - [OpenClaw Harness](#openclaw-harness)
  - [Automated Research Pipelines](#automated-research-pipelines)
  - [Fee Proposal Generator](#fee-proposal-generator)
  - [Docx Styler](#docx-styler)
  - [SHA-SG](#sha-sg)
- [Design Philosophy](#design-philosophy)
- [About](#about)

---

## Portfolio at a Glance

| Metric | Detail |
|---|---|
| **Combined codebase** | 140,000+ lines (Python, JavaScript, HTML/CSS, Markdown) |
| **Total commits** | 1,600+ across all projects |
| **Production files** | 750+ tracked files (excluding knowledge base content) |
| **First commit** | 2023 |
| **Stack** | Python · Direct LLM API integration (Gemini, Claude, GPT, MiniMax) · Custom prompt orchestration · Full deployment pipelines |

---

## Projects

### [Translation Pipeline](./translation-pipeline/)
**A complete AI-assisted system built specifically for legal and regulatory document translation** · *In active use*

- **Multi-model routing** — separate models for OCR correction, table reconstruction, and translation, each with task-appropriate temperature tuning
- **Language-aware prompt architecture** — specialized prompts where linguistically necessary (Thai, Bengali), generic prompts where not
- **Anti-runaway chunking** — production-hardened protections against LLM content expansion and infinite recursion in long documents
- **Hybrid extraction routing** — born-digital PDFs go through local native extraction with page-level layout classification; scanned documents and raster images route to a switchable cloud OCR lane (currently, Google Document AI or Datalab)
- **Partner API** — server-to-server REST API with bearer-token auth, webhook delivery with HMAC signatures, idempotency keys, and per-partner rate and concurrency limits
- **Review-aware output** — structured findings identify where human inspection is needed, rather than silently passing risky output
- **Inputs:** PDF, DOCX, images, public webpages
- **Outputs:** Markdown, DOCX, HTML, PDF
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
           └── Text cleanup 🤖

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

The LKB is the authoritative legal source layer for the system: ingestion, versioned Markdown, provenance, validation, audit trail, and SQLite-backed search.

- **Planned upstream source:** the [Translation Pipeline](./translation-pipeline/), whose structure-corrected translated output is designed to flow directly into LKB ingestion
- **Current status:** ingestion, validation, UI-assisted review, canonical storage, and search indexing are operational
- **Downstream role:** the LKB provides the source-backed citations that the Legal Wiki uses for synthesis pages

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

### Legal Wiki
**Synthesis-only legal wiki layer** · *In active development*

The LW turns source-backed LKB materials into living synthesis pages: topics, issues, concepts, timelines, entities, and questions.

- **Authority boundary:** the LW is not canonical legal authority; it summarizes and compares the LKB
- **Source grounding:** pages carry LKB references and use citation-backed excerpts rather than duplicating full legal text
- **Authoring flow:** page helpers support both blank page creation and source-grounded draft generation, with human review still required

---

### [OpenClaw Harness](./openclaw-harness/)
**LLM-to-OS bridge for local task execution** · *In active use*

OpenClaw connects LLM reasoning to a local execution runtime, enabling automated task execution through filesystem operations, shell commands, application control, and network requests. This deployment integrates multiple models (MiniMax 2.5 Pro for routine operations, Claude Sonnet for complex reasoning) with explicit routing logic. LobsterBoard provides the operational dashboard for managing mini-apps, scheduled jobs, monitoring execution, and system analytics.

<div align="center">
  <img src="./openclaw-harness/screenshots/ocr-dashboard.png" alt="OCR Dashboard" width="600">
  <br><em>OCR Dashboard — Multi-engine document processing with ASEAN language support</em>
  <br><sub>As of 18 March 2026</sub>
</div>

<p>&nbsp;</p>

→ [Architecture & Design](./openclaw-harness/README.md)

---

### [Automated Research Pipelines](./automated-research-pipelines/)
**Scheduled AI-driven research, summarization, and distribution** · *In active use*

A set of autonomous pipelines running on cron schedules via OpenClaw. Each pipeline spawns an isolated agent session, performs targeted web research, produces LLM-generated summaries, and distributes results to Discord channels and dashboards. Currently runs daily news briefings and an AI legal tech jobs/funding tracker.

<div align="center">
  <img src="./automated-research-pipelines/screenshots/lobsterboard.png" alt="LobsterBoard Dashboard" width="600">
  <br><em>LobsterBoard — Cron schedules, activity log, system monitoring, and live sessions</em>
  <br><sub>As of 18 March 2026</sub>
</div>

<p>&nbsp;</p>

→ [Architecture & Design](./automated-research-pipelines/README.md)

---

### [Fee Proposal Generator](./fee-proposal-generator/)
**Automated fee proposal drafting for legal engagements** · *Retired — replaced by firm-wide legal AI platform*

Generates structured fee proposals from intake parameters — scope, personnel, billing/deposit requirements. Reduces a 2–3 hour manual drafting process to minutes while maintaining firm-specific formatting and compliance with engagement standards.

→ [Architecture & Design](./fee-proposal-generator/README.md)

---

### [Docx Styler](./docx-styler/)
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

- **Legal-native data models.** Clause hierarchies, numbered sections, mixed numeral systems, and bilingual instrument pairs are modeled as first-class structures, not flattened into plain text.

- **Review-aware over silently confident.** Output that looks correct but has hidden defects is a serious risk for legal work. These systems surface uncertainty rather than hide it — structured review findings and explicit warnings are first-class features.

---

## About

- **Background:** U.S.-qualified technology lawyer at DFDL Legal & Tax
- **AI leadership roles:** AI Strategic Initiative committee member, regional lead for Knowledge Managers, and Legal AI Workflows Champion
- **Builder:** designer and developer of the AI systems documented here — 80,000+ lines of production code across legal translation, knowledge management, document automation, and workflow orchestration
- **Region:** based in ASEAN, working across Thai, Singaporean, and regional legal frameworks

For more information: [emsato.com](https://emsato.com) or [LinkedIn](https://www.linkedin.com/in/emsato/)

---

*Last updated: 30 March 2026*
