# Legal AI Projects

System architectures and design documentation for legal AI tools built for real-world legal practice — covering contract analysis, regulatory knowledge systems, workflow automation, and retrieval-augmented generation (RAG) pipelines.

**Built by a U.S.-qualified technology lawyer working across ASEAN jurisdictions.**

> This repository documents the architecture, design decisions, and system flows behind a portfolio of legal AI tools currently in production or active development. Source code is maintained in private repositories. What follows is the engineering reasoning — the *why* and *how* behind each system.

---

## At a Glance

| Metric | |
|---|---|
| **Combined codebase** | ~71,000 lines (Python, JavaScript, HTML/CSS, Markdown) |
| **Total commits** | 1,035+ across all projects |
| **Production files** | 198 tracked files |
| **First commit** | March 2025 |
| **Stack** | Python · Direct LLM API integration (Gemini, Claude, GPT, MiniMax) · Custom prompt orchestration · Full deployment pipelines |

---

## Projects

### [Translation Pipeline](./translation-pipeline/)
**Fully automated legal document translation system**

Production system for translating long-form legal documents across ASEAN languages (Thai, Vietnamese, Bahasa, and others). Handles PDF ingestion, structural preservation, self-correcting loops, and tunable output quality. Built for *low- or under-resourced language* documents that demand precision—statutes, regulations, technical guides—often found only in difficult formats like lengthy, scanned PDFs.

→ [Architecture & Design](./translation-pipeline/README.md)

---

### [OpenClaw Harness](./openclaw-harness/)
**LLM-to-OS bridge for local task execution**

Three-component system that connects large language models to the local operating system for real-world task execution. Routes tasks between models based on complexity (MiniMax 2.5 Pro for routine operations, Claude Sonnet for complex reasoning). Includes a dashboard for managing automated mini-applications, scheduled jobs, and system analytics.

→ [Architecture & Design](./openclaw-harness/README.md)

---

### [Legal Knowledge Base](./legal-knowledge-base/)
**Files-first ingestion system for legal RAG**

Ingestion and chunking pipeline for primary legal regulations, secondary legal resources, templates, and practice notes — designed as the foundation for a retrieval-augmented generation system. Currently operational for document processing and structured storage; RAG retrieval layer is in active development.

→ [Architecture & Design](./legal-knowledge-base/README.md)

---

### [Legal Jobs Dashboard](./legal-jobs-dashboard/)
**Self-updating market intelligence for AI + legal roles**

Automated dashboard that monitors, aggregates, and analyzes AI-related legal job postings. Tracks employer patterns, role requirements, compensation signals, and market trends. Runs on scheduled pipelines with minimal manual intervention.

→ [Architecture & Design](./legal-jobs-dashboard/README.md)

---

### [Fee Proposal Generator](./fee-proposal-generator/)
**Automated fee proposal drafting for legal engagements**

Generates structured fee proposals from intake parameters — scope, personnel, billing/deposit requirements. Reduces a 2–3 hour manual drafting process to minutes while maintaining firm-specific formatting and compliance with engagement standards.

→ [Architecture & Design](./fee-proposal-generator/README.md)

---

### [Docx Styler](./docx-styler/)
**AI-driven paragraph styling for Word documents**

Applies consistent paragraph-level styling to legal Word documents using AI classification. Solves the endemic problem of inconsistent formatting in documents that pass through multiple hands — associates, partners, clients, opposing counsel — before final production.

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

U.S.-qualified technology lawyer at DFDL Legal & Tax, where I serve on the AI Strategic Initiative committee, lead Knowledge Management, and champion firm-wide workflow automation. I design and build the AI systems documented here — 80,000+ lines of production code across legal translation, knowledge management, document automation, and workflow orchestration. Based in ASEAN, where I work across Thai, Singapore, and regional legal frameworks.

For more: [emsato.com](https://emsato.com)/[LinkedIn](https://www.linkedin.com/in/emsato/)

---

*Last updated: 17 March 2026*
