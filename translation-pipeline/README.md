# Translation Pipeline

**AI-assisted legal and regulatory document translation workflow**

This project is a production-oriented translation pipeline for long-form legal and regulatory documents. It is designed for cases where ordinary machine translation is not enough because the document structure matters as much as the wording: numbered clauses, tables, notes, headings, cross-references, and formatting all need to survive the workflow.

The system is built around a 4-step pipeline:

1. **Convert** source material into Markdown
2. **Correct** OCR and structural issues
3. **Translate** with language-pair-specific prompts and term handling
4. **Export** to final formats such as DOCX, HTML, and PDF

**Live app (development server):** [dev.translation.legal](https://dev.translation.legal)

---

## Screenshots

<div align="center" style="margin-bottom:2em">
  <img src="./screenshots/splash.png" alt="Translation Pipeline — Home" width="700">
  <br><em>Home — Quick Translation and Advanced Mode entry points</em>
  <br><sub>As of 18 March 2026</sub>
</div>

<div align="center" style="margin-bottom:2em">
  <img src="./screenshots/advanced-mode.png" alt="Translation Pipeline — Advanced Mode" width="700">
  <br><em>Advanced Mode — Step-by-step pipeline control with per-stage review</em>
  <br><sub>As of 18 March 2026</sub>
</div>

<div align="center">
  <img src="./screenshots/task-dashboard.png" alt="Translation Pipeline — Task Dashboard" width="700">
  <br><em>Task Dashboard — Production usage tracking across translation jobs</em>
  <br><sub>As of 18 March 2026</sub>
</div>

---

## What This Project Is Solving

General-purpose translation tools often perform reasonably well on short text, but legal documents introduce a different class of problems:

- Clause numbering can drift or flatten
- Tables can collapse into plain prose
- Notes and cross-references can move or disappear
- Key legal terms can be translated inconsistently across a long document
- OCR noise can silently propagate into the final output

This pipeline is built to reduce those risks and to surface them when they cannot be fully resolved automatically.

---

## Current Product Shape

The current live system is strongest on:

- **Born-digital PDFs**
- **Public HTML webpages**
- **English hub translation flows**, especially **English ↔ Thai**

Supported output formats include:

- Markdown
- DOCX
- HTML
- PDF

Current language status:

- **Most mature:** English ↔ Thai
- **Also supported in the live app:** English ↔ Indonesian, Malay, Vietnamese, Simplified Chinese
- **Still under review:** Bengali, Lao, Khmer, Burmese

Important limitations:

- **Scanned PDFs are deployment-dependent.** On lower-tier servers they may be gated off or flagged for manual review.
- **Complex layouts still need review.** Tables, diagrams, footnotes, and multi-column pages are much better handled than before, but they are still not risk-free.
- **URL input is for public webpages, not full website crawling.** The app can extract a page, and for some homepage-style URLs it may follow a small number of same-site links, but it does not guarantee full multi-page website coverage.

---

## Pipeline Overview

### Step 1: Convert to Markdown

The system accepts uploaded PDFs and public webpage URLs.

For PDFs, the current live path uses native text extraction first and falls back to OCR only when needed. For public webpages, the app extracts the main readable content and converts it to Markdown. Step 1 also adds review signals when the source appears risky, for example:

- scanned-page OCR
- ambiguous layout
- likely JavaScript-dependent or login-gated webpages
- graphic-heavy or multi-column pages

### Step 2: Correct and Normalize

This stage prepares the Markdown for translation by addressing:

- OCR mistakes
- table damage
- numbering drift
- footnote/endnote issues
- structural cleanup

It now also emits structured review findings when the output looks materially different from the source in ways that could matter legally.

### Step 3: Translate

Translation is not treated as a single blind pass. The pipeline applies:

- language-pair-specific prompt templates
- critical-term preservation
- deterministic substitutions where needed
- post-translation review checks

The goal is not just fluent output, but legally usable output with better consistency across a full document.

### Step 4: Export

Final output can be generated as:

- DOCX
- HTML
- PDF

When review signals matter, they can carry forward into the final output so the document itself makes clear that manual review is still required.

---

## Review-Aware Design

One of the core ideas behind the project is that not every risky document should silently pass as “done.”

Instead of pretending the pipeline is infallible, the system can mark a result as:

- **review required**
- accompanied by **warnings**
- accompanied by **specific findings or targets to inspect**

That review information appears in the status UI and, where appropriate, in the final exported document.

This is especially important for:

- scanned Thai documents
- layout-heavy PDFs
- structurally ambiguous extractions
- documents where numbering, notes, or tables may have shifted

---

## Architecture in Practice

At a high level, the project is a web application backed by asynchronous processing:

- **Frontend:** browser-based Advanced Mode and Quick Translation workflows
- **Backend:** FastAPI
- **Task processing:** Celery-based step execution and orchestration
- **Document conversion:** Pandoc plus custom formatting logic
- **LLM providers:** provider and language pair-specific prompt paths for correction and translation
- **Monitoring/admin:** task status UI, admin routes, health checks, and error monitoring

---

## Why This Is Different from a Generic Translation Wrapper

This project is not just “upload a file, call an API, return text.”

The pipeline adds value in the areas that matter most for legal and regulatory documents:

- preserving structure before translation
- correcting OCR and layout issues before they become translation errors
- carrying review signals through the pipeline
- producing final deliverables in business-ready formats

The real differentiator is not that it eliminates all review. It is that it tries to preserve the parts of legal documents that are easiest to damage, and it signals clearly when the output should not be trusted without inspection.

---

## Status

This is an active production project, not a finished research artifact.

The strongest current story is:

- born-digital legal/regulatory PDFs
- English-hub translation flows
- review-aware output for high-risk cases

The biggest remaining technical gap is still the hardest class of input:

- scanned, layout-heavy, multilingual documents on constrained infrastructure

---

*→ Back to [Project Index](../README.md)*
