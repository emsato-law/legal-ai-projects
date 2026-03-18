# Legal AI Projects

System architectures and design documentation for legal AI tools built for real-world legal practice — regulatory knowledge systems, workflow automation, and retrieval-augmented generation (RAG) pipelines.

**Built by a U.S.-qualified technology lawyer working across ASEAN jurisdictions.**

This repository documents the architecture, design decisions, and system flows behind a portfolio of legal AI tools currently in production or active development. Source code is maintained in private repositories. What follows is the engineering reasoning — the *why* and *how* behind each system.

---

## At a Glance (for this portfolio)

| Metric | |
|---|---|
| **Combined codebase** | 80,000+ lines (Python, JavaScript, HTML/CSS, Markdown) |
| **Total commits** | 1,500+ across all projects |
| **Production files** | 250+ tracked files |
| **First commit** | 2023 |
| **Stack** | Python · Direct LLM API integration (Gemini, Claude, GPT, MiniMax) · Custom prompt orchestration · Full deployment pipelines |

---

## Projects

### [Translation Pipeline](./translation-pipeline/)
**Fully automated legal document translation system**

Production-oriented translation pipeline for long-form legal and regulatory documents across English-hub ASEAN language pairs, especially Thai, Vietnamese, Indonesian, and Malay. Handles PDF and public-webpage ingestion, structure-aware preprocessing, review-aware correction, and export to business-ready formats. Built for documents where precision matters more than fluency—statutes, regulations, and technical guidance that often arrive as complex PDFs and other difficult source formats.

<p align="center">
  <img src="./translation-pipeline/screenshots/splash.png" alt="Translation Pipeline — Home" width="600">
  <br><em>Home — Quick Translation and Advanced Mode entry points</em>
</p>
<p align="center"><sub>As of 18 March 2026</sub></p>
<br><br>

<p align="center">
  <img src="./translation-pipeline/screenshots/advanced-mode.png" alt="Translation Pipeline — Advanced Mode" width="600">
  <br><em>Advanced Mode — Step-by-step pipeline control with per-stage review</em>
</p>
<p align="center"><sub>As of 18 March 2026</sub></p>
<br><br>

<p align="center">
  <img src="./translation-pipeline/screenshots/task-dashboard.png" alt="Translation Pipeline — Task Dashboard" width="600">
  <br><em>Task Dashboard — Production usage tracking across translation jobs</em>
</p>
<p align="center"><sub>As of 18 March 2026</sub></p>

→ [Architecture & Design](./translation-pipeline/README.md)

---

### [OpenClaw Harness](./openclaw-harness/)
**LLM-to-OS bridge for local task execution**

OpenClaw connects LLM reasoning to a local execution runtime, enabling automated task execution through filesystem operations, shell commands, application control, and network requests. This deployment integrates multiple models (MiniMax 2.5 Pro for routine operations, Claude Sonnet for complex reasoning) with explicit routing logic. LobsterBoard provides the operational dashboard for managing mini-apps, scheduled jobs, monitoring execution, and system analytics.

<p align="center">
  <img src="./openclaw-harness/screenshots/ocr-dashboard.png" alt="OCR Dashboard" width="600">
  <br><em>OCR Dashboard — Multi-engine document processing with ASEAN language support</em>
</p>
<p align="center"><sub>As of 18 March 2026</sub></p>

→ [Architecture & Design](./openclaw-harness/README.md)

---

### [Legal Knowledge Base](./legal-knowledge-base/)
**Files-first ingestion system for legal RAG**

Ingestion and chunking pipeline for primary legal regulations, secondary legal resources, templates, and practice notes — designed as the foundation for a retrieval-augmented generation system. Currently operational for document processing and structured storage; RAG retrieval layer is in active development.

→ [Architecture & Design](./legal-knowledge-base/README.md)

---

### [Automated Research Pipelines](./automated-research-pipelines/)
**Scheduled AI-driven research, summarization, and distribution**

A set of autonomous pipelines running on cron schedules via OpenClaw. Each pipeline spawns an isolated agent session, performs targeted web research, produces LLM-generated summaries, and distributes results to Discord channels and dashboards. Currently runs daily news briefings and an AI legal tech jobs/funding tracker.

<p align="center">
  <img src="./automated-research-pipelines/screenshots/lobsterboard.png" alt="LobsterBoard Dashboard" width="600">
  <br><em>LobsterBoard — Cron schedules, activity log, system monitoring, and live sessions</em>
</p>
<p align="center"><sub>As of 18 March 2026</sub></p>

→ [Architecture & Design](./automated-research-pipelines/README.md)

---

### [Fee Proposal Generator](./fee-proposal-generator/)
**Automated fee proposal drafting for legal engagements**

Generates structured fee proposals from intake parameters — scope, personnel, billing/deposit requirements. Reduces a 2–3 hour manual drafting process to minutes while maintaining firm-specific formatting and compliance with engagement standards.

→ [Architecture & Design](./fee-proposal-generator/README.md)

---

### [Docx Styler](./docx-styler/)
**AI-driven paragraph styling for Word documents**

Applies consistent paragraph-level styling to legal Word documents using AI classification. Addresses the endemic problem of inconsistent formatting in documents that pass through multiple hands — associates, partners, clients, opposing counsel — before final production.

→ [Architecture & Design](./docx-styler/README.md)

---

### [SHA-SG](./sha-sg/)
**Singapore venture capital templates under version control**

Places standard Singapore law venture capital template agreements (shareholders' agreement, subscription agreement, constitution) into structured version control. Tracks clause-level changes across template revisions and enables diffing between template versions.

→ [Architecture & Design](./sha-sg/README.md)

---

## Design Philosophy

These systems share a common set of architectural principles:

- **Direct API integration over frameworks.** No LangChain, no LlamaIndex. Every LLM call is a deliberate, auditable API call with custom prompt management. This trades convenience for control — essential when outputs carry legal consequences.

- **Model routing by task complexity.** Lightweight models handle routine extraction and classification. Capable models handle reasoning, drafting, and ambiguous interpretation. Cost and latency stay proportional to difficulty.

- **Legal-native data models.** Document structures, metadata schemas, and output formats are designed around how lawyers actually work — not retrofitted from general-purpose AI tooling.

- **Deployment where it matters.** Production systems ship with full deployment pipelines. Smaller tools (Docx Styler, SHA-SG) are designed as local utilities — deployed to the environments where they're actually used.

---

## About

U.S.-qualified technology lawyer at DFDL Legal & Tax, where I serve on the AI Strategic Initiative committee, lead Knowledge Management, and champion firm-wide workflow automation. I design and build the AI systems documented here — 80,000+ lines of production code across legal translation, knowledge management, document automation, and workflow orchestration. Based in ASEAN, where I work across Thai, Singaporean, and regional legal frameworks.

For more: [emsato.com](https://emsato.com) or [LinkedIn](https://www.linkedin.com/in/emsato/)

---

*Last updated: 18 March 2026*
