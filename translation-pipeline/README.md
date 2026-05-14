# Translation Pipeline

**A complete AI-assisted system built specifically for legal and regulatory document translation**

---

**Current app host:** [translate.emsato.com](https://translate.emsato.com)

---

## Table of Contents

- [What This Project Is Solving](#what-this-project-is-solving)
- [Architecture in Practice](#architecture-in-practice)
  - [System Stack](#system-stack)
  - [Current Product Shape](#current-product-shape)
- [Pipeline Overview](#pipeline-overview)
  - [Step 1: Convert to Markdown](#step-1-convert-to-markdown)
  - [Step 2: Correct and Normalize](#step-2-correct-and-normalize)
  - [Step 1 to Step 2 Diff Viewer](#step-1-to-step-2-diff-viewer)
  - [Step 3: Translate](#step-3-translate)
  - [Step 4: Export](#step-4-export)
  - [Partner API](#partner-api)
- [What Makes This Different](#what-makes-this-different)
  - [Model Routing](#model-routing)
  - [Language-Aware Prompt Architecture](#language-aware-prompt-architecture)
  - [Chunking Safety](#chunking-safety)
  - [Large Document Handling](#large-document-handling)
  - [Quality Testing Infrastructure](#quality-testing-infrastructure)
  - [Review-Aware Design](#review-aware-design)
- [Status](#status)
  - [Latest Developments](#latest-developments)
  - [Future Developments](#future-developments)

---

## What This Project Is Solving

General-purpose translation tools can produce a usable rough draft, but fall short of professional standards when the output is client-facing. This system addresses the specific failure modes that matter in legal documents in this region:

- **Long documents** — General AI translation tools may summarize, hallucinate, or fail outright, especially with scanned PDFs
- **Term consistency** — Key legal terms are translated differently across a long document, undermining precision
- **Low-resource languages** — Poor performance or no support for regional languages common in ASEAN legal work
- **Blind output** — Results cannot be trusted without review, but provide no guidance on where to look
- **Document structure** — Clause numbering, tables, charts, footnotes, and cross-references are poorly preserved or silently lost

This system is built to reduce those risks and to surface them when they cannot be fully resolved automatically.

---

## Architecture in Practice

At a high level, the project is a web application backed by asynchronous processing:

<div align="center">
  <img src="./screenshots/splash.png" alt="Translation Pipeline — Home" width="700">
  <br><em>Home — Quick Translation and Advanced Mode entry points</em>
  <br><sub>As of 18 March 2026</sub>
</div>

<p>&nbsp;</p>

<div align="center">
  <img src="./screenshots/advanced-mode.png" alt="Translation Pipeline — Advanced Mode" width="700">
  <br><em>Advanced Mode — Step-by-step pipeline control with per-stage review</em>
  <br><sub>As of 18 March 2026</sub>
</div>

<p>&nbsp;</p>

<div align="center">
  <img src="./screenshots/task-dashboard.png" alt="Translation Pipeline — Task Dashboard" width="700">
  <br><em>Task Dashboard — Production usage tracking across translation jobs</em>
  <br><sub>As of 18 March 2026</sub>
</div>

<p>&nbsp;</p>

### System Stack

- **Frontend:** browser-based Quick Translation, Advanced Mode, Check Status with stable job IDs and task-log viewing, Step 1 to Step 2 cleanup diff, and dashboard workflows
- **Web control plane:** FastAPI served from a managed cloud web runtime
- **Long-running execution:** queued document jobs dispatched to isolated worker runs for pipeline processing
- **State and artifacts:** durable job state plus retained source, intermediate, final, and task-log artifacts
- **Document conversion:** Pandoc with tiered memory limits and graduated timeouts
- **LLM providers:** Gemini 2.5 Pro, GPT-5-mini, Claude Sonnet 4 — provider and language pair-specific prompt paths
- **Partner API:** REST API (`/api/v1/`) with bearer-token auth, webhook delivery, idempotency, partial workflows, cancellation, purge, and per-partner rate and concurrency limits
- **Periodic maintenance:** automated stuck-task recovery, orphaned file cleanup, and database pruning via scheduled background tasks
- **Admin dashboard:** production task monitoring, partner operations, artifact inspection, and support analysis — with protected access
- **Error monitoring:** Sentry integration for real-time error tracking across the web control plane and worker runtime
- **Three-tier logging:** user-facing logs in the dashboard UI, technical debug logs for developers, and infrastructure logs for operations — each with its own storage and audience
- **Deployment:** managed web control plane plus isolated worker runtime, cache-busted static bundles, and secret-managed configuration
- **Security:** authenticated access, content-aware upload validation, app-local rate limiting on upload endpoints, and scoped partner credentials

### Current Product Shape

```
Inputs                         Languages                      Outputs
─────────────────────          ─────────────────────          ─────────────────────
DOCX                           English ↔ Thai (most mature)   Markdown
Born-digital PDF               English ↔ Indonesian           DOCX
Scanned PDF                    English ↔ Malay                HTML
Raster images (PNG, JPG, etc.) English ↔ Vietnamese           PDF
Public webpage URLs            English ↔ Simplified Chinese
                               English ↔ Bengali
                               English ↔ Lao
                               English ↔ Khmer
                               English ↔ Burmese
```

All language pairs use English as the hub language. English ↔ Thai is the most mature pair with the deepest prompt tuning and production usage.

---

## Pipeline Overview

The system is built around a 4-step pipeline:

1. **Convert** source material into Markdown
2. **Correct** OCR and structural issues
3. **Translate** with language-pair-specific prompts and term handling
4. **Export** to final formats such as Markdown, DOCX, HTML, and PDF

Advanced Mode also exposes the handoff between Step 1 and Step 2. Reviewers can compare raw conversion Markdown against corrected/normalized Markdown before trusting the document for translation or export.

### Step 1: Convert to Markdown

The system accepts uploaded documents, images, and public webpage URLs. Rather than using a single extraction path, Step 1 routes each input to the provider best suited to its characteristics:

- **DOCX and born-digital PDFs** (with local-safe languages) go through native extraction with page-level layout classification. Each page is categorized — table, narrative, multi-column, graphic/diagram, or ambiguous — and the extraction strategy adjusts accordingly. The native route preserves common footnote and endnote structures, provisional tables, and born-digital table layouts before the cleanup step begins.
- **Scanned documents and images** (scanned PDFs, weak-text PDFs, non-local-safe languages, raster image uploads) route to a switchable OCR lane — currently, Google Document AI or Datalab, selected per deployment as an operational decision rather than a fixed architectural choice.
- **Public webpage URLs** go through HTML extraction and Markdown conversion.

A planned addition to Step 1 is **sensitive information anonymization** — stripping personally identifiable and confidential content before documents are sent to cloud OCR or downstream LLM-powered steps. This ensures sensitive information never leaves the local machine. Anonymized content is re-inserted after translation in Step 3.

Native extraction, cloud OCR, and URL conversion surface review signals when layout fidelity is uncertain. Scanned Thai pages, ambiguous layout, graphic-heavy or multi-column content, suspected legal-reference drift, incomplete legal dates, suspicious amount phrases, and likely JavaScript-dependent or login-gated webpages are all flagged for downstream awareness.

### Step 2: Correct and Normalize

This is not a single AI call. Step 2 runs a multi-stage pipeline where each stage addresses a specific class of legal document damage:

1. **OCR Correction** — Repairs character-level errors from PDF extraction, with language-specific awareness (e.g., Thai script patterns, Bengali conjunct consonants)
2. **Table Reconstruction** — Detects and repairs damaged table structures, including grid tables that survived extraction as partial fragments
3. **Numbering Validation** — Verifies hierarchical clause numbering hasn't drifted, supporting mixed numeral systems (Thai ๑๒๓, Bengali ০১২, Arabic 123 in the same document)
4. **Structural Analysis** — Emits structured review findings when the output shows signs of material loss: table damage, likely row loss, heading flattening, missing note labels, numeral drift, low structural similarity, or severe content shrinkage
5. **Text Cleanup** — Language-aware final normalization

Each sub-stage is independently configurable and can be enabled or disabled per deployment. The processing is language-aware where it matters (Thai numerals, Bengali script, Thai legal references) and universal where it doesn't (footnotes, numbering hierarchies).

When both Step 1 and Step 2 artifacts are available, Advanced Mode can also show an on-demand cleanup diff so reviewers can see what changed during OCR cleanup and structural normalization.

### Step 1 to Step 2 Diff Viewer

The newest review surface is a cleanup diff viewer between the conversion artifact and the corrected artifact. It is designed for the exact point where document risk is highest: after extraction, before translation.

The viewer helps reviewers inspect:

- OCR repair and text normalization changes
- table reconstruction and possible row/column shifts
- heading, footnote, endnote, and clause-numbering changes
- material additions, removals, or shrinkage between extracted and corrected Markdown
- whether Step 2 improved the document or introduced a review issue before downstream translation

Because the app retains per-step artifacts, this comparison is available on demand from Advanced Mode and related job views rather than being buried in logs or final output.

### Step 3: Translate

Translation is not treated as a single blind pass. The pipeline applies:

- language-pair-specific prompt templates
- critical-term preservation
- deterministic substitutions where needed
- post-translation review checks
- re-insertion of sensitive information (planned — corresponding to the anonymization step in Step 1)

For Thai-to-English work, the system now keeps narrow legal markers deterministic while leaving terminology and style choices to prompt guidance and review warnings. The goal is not just fluent output, but legally usable output with better consistency across a full document.

### Step 4: Export

Final output can be generated as:

- Markdown (also the format expected by the Legal Knowledge Base for downstream ingestion)
- DOCX
- HTML
- PDF

When review signals matter, they can carry forward into the final output so the document itself makes clear that manual review is still required. Final documents can include a review-required banner, and dashboard/status views expose the underlying warnings, review targets, and structured findings.

The longer-term goal is for translation pipeline outputs to feed directly into the [Legal Knowledge Base](../legal-knowledge-base/) — translated and structure-corrected documents becoming ingestion-ready source material for the canonical legal layer without manual reformatting. From there, the Legal Wiki can build source-grounded synthesis on top of Legal Knowledge Base citations.

### Partner API

The system exposes a server-to-server REST API for programmatic access to the full translation pipeline:

- **Authentication:** bearer-token API keys with per-partner rate limits and concurrent job caps
- **Job lifecycle:** create jobs from file uploads or public URLs, poll for status, list job events, cancel active jobs, and purge terminal jobs
- **Partial workflows:** partners can start from selected pipeline stages and retrieve per-step artifacts instead of always running the full 4-step pipeline
- **Artifacts:** source, intermediate, final, and task-log artifacts can be listed and downloaded through scoped job endpoints
- **Webhook delivery:** terminal events (`job.completed`, `job.failed`, `job.cancelled`) are pushed to partner-supplied URLs with HMAC-SHA256 signatures for verification
- **Idempotency:** safe retries via `Idempotency-Key` headers — duplicate submissions return the original job instead of creating a new one
- **Review metadata:** `warnings`, `review_findings`, and `review_targets` from the pipeline are surfaced directly in API responses, so downstream systems can programmatically decide whether output needs human review
- **Self-describing:** OpenAPI schema at `/openapi.json`, interactive docs at `/docs`, and a public instructions page at `/api-docs`

This is how the pipeline is designed to integrate with other systems — a law firm's document management platform, a knowledge base ingestion service, or a client-facing portal can submit jobs and receive results without interacting with the web UI.

---

## What Makes This Different

The system is not a translation API wrapper. It corrects document damage before translating, routes each task to the right model, and tells reviewers exactly where to look when output cannot be fully trusted.

### Model Routing

The pipeline does not use a single model for everything. Each processing stage uses the model best suited to its constraints:

- **Step 2 (Correction):** GPT-5-mini at temperature 0.0 — deterministic output for OCR repair, table reconstruction, and numbering validation where creativity is a liability
- **Step 3 (Translation):** Gemini 2.5 Pro at temperature 0.1 — high token capacity (65K) for long legal documents with minimal creative drift
- **Fallback translation:** Claude Sonnet 4 with reduced chunk sizes to accommodate its lower output ceiling (4K tokens)

Token budgets, chunk sizes, and expansion ratios are tuned per provider and per language pair — a Thai→English translation under Gemini gets different chunking than the same pair under Claude.

The temperature choices are legally motivated: OCR correction should never be creative, and translation should be as deterministic as possible while still reading naturally.

### Language-Aware Prompt Architecture

Not every language gets a custom prompt. The pipeline distinguishes between languages that need specialized handling and those that don't:

- **Thai and Bengali** use dedicated correction and translation prompts — Thai because of mixed numeral systems (Thai ๑๒๓ alongside Arabic 123, Buddhist Era dates), Bengali because of its own numerals and complex conjunct consonants
- **Malay, Indonesian, and Vietnamese** use generic prompts because they share Latin script, Arabic numerals, and space-separated word boundaries with English — custom prompts would add cost without benefit

This is a deliberate cost-benefit decision, not a gap. The prompt loading system automatically falls back to generic prompts when no language-specific variant exists.

### Chunking Safety

Legal documents routinely exceed LLM token limits, so the pipeline splits them into chunks. But naive chunking creates a dangerous failure mode: when an LLM receives a small chunk, it can try to "complete" the document, generating massive repetitive content that triggers infinite re-splitting.

The pipeline includes production-hardened protections against this:

- **Dynamic instruction injection:** Single-chunk documents get full-document processing instructions; multi-chunk documents get explicit "process only this section" constraints
- **Expansion detection:** Responses more than 3× larger than their input are flagged as bloated and discarded
- **Recursion limits:** Maximum 2 levels of chunk splitting, preventing infinite loops
- **Size-based protection:** Chunks below 1KB are never split further

These protections were built in response to a real production incident where Bengali documents triggered 70× content expansion and infinite recursion.

### Large Document Handling

Long documents create compounding risks — more chunks mean more chances for a single failure to stall the pipeline. The system includes specific protections for large documents:

- **Per-chunk timeouts:** each chunk has a 20-minute wall-clock limit, cumulative across retries, so a single problematic chunk cannot consume the full step timeout
- **Automatic stage skipping:** documents with 20 or more chunks skip numbering validation (Step 2c) — a deliberate tradeoff that reduces processing time significantly while preserving the stages with the highest impact
- **Tiered step timeouts:** Step 2 and Step 3 get 8-hour timeouts; Step 1 and Step 4 get 1-hour timeouts — reflecting the actual time profiles of each stage
- **Tiered export handling:** final conversion uses graduated memory limits and timeouts based on output file size, with automatic format fallback for very large files

### Quality Testing Infrastructure

Translation quality cannot be verified by unit tests alone. The project includes a structured testing framework for tracking real-document quality across languages, OCR routes, and document characteristics:

- **Quality matrix:** a coverage snapshot tracking pass/warn/fail status per language, per OCR route, and per document challenge factor (clean, scanned, diagram-heavy, multi-column, footnotes, etc.)
- **Campaign-based testing:** a defined sweep order — calibration on Thai, then baseline across all languages, then stress tests for progressively harder document types
- **Synthetic benchmark:** generates born-digital PDF fixtures and scores table/footnote/endnote preservation, runnable locally before any Step 1 structural change
- **Gold-corpus benchmark:** compares real challenge PDFs against manually verified Gold Markdown, with automatic slicing for over-limit documents, candidate review reports, and frozen fixture manifests

### Review-Aware Design

One of the core ideas behind the project is that not every risky document should silently pass as "done."

Instead of pretending the pipeline is infallible, the system emits structured signals that identify *what* to inspect:

- **`review_targets`:** specific pages or sections flagged for human attention
- **`review_findings`:** categorized issues such as likely table loss, note loss, numeral drift, or severe content shrinkage
- **`warnings`:** contextual notes about source quality (e.g., scanned page, JavaScript-dependent webpage)

These signals are not just boolean flags. They propagate through every pipeline step and into the final exported document, so a reviewer receiving a translated DOCX knows not just *that* review is needed, but *where to look*.

This is especially important for:

- scanned Thai documents
- layout-heavy PDFs
- structurally ambiguous extractions
- documents where numbering, notes, or tables may have shifted

The review surface is also becoming more inspectable. Status pages and the dashboard expose review badges and filtered views, the app links to warning guidance, and Advanced Mode includes a Step 1 to Step 2 cleanup diff viewer so reviewers can inspect normalization changes before relying on the translation.

---

## Status

This is an active production system, in daily use for the builder's legal practice.

### Latest Developments

- **Managed worker architecture** — the production path now separates the web control plane from long-running worker jobs, with durable state and retained per-step artifacts
- **Partner and partial-workflow APIs** — server-to-server access now covers full jobs, selected step ranges, scoped artifacts, webhooks, cancellation, purge, and idempotent retries
- **Switchable OCR routing** — hybrid Step 1 routing supports local/native extraction for safe born-digital inputs and cloud OCR for harder scans and images
- **Thai OCR and legal-reference review** — scanned Thai pages now carry explicit review-required signals, legal-reference drift targets, incomplete-date warnings, and suspicious-amount review flags
- **Step 1 to Step 2 diff viewer** — Advanced Mode can compare conversion output against corrected/normalized Markdown before translation or export
- **Operator support surfaces** — Check Status now exposes stable job IDs, linked task-log viewing, and better handoff between status pages, retained artifacts, and dashboard review
- **Gold-corpus QA** — real challenge PDFs are tracked against curated Gold Markdown, with candidate review reports before fixture changes
- **TH-EN preserve rules** — Thai-to-English translation now protects narrow legal markers and selected formal references without broad semantic rewrites
- **Warning guidance** — status pages, final documents, emails, and warning reference pages now present clearer guidance for manual review

### Future Developments

- Sensitive information anonymization before cloud OCR and LLM processing
- Expanded production testing across all supported language pairs (Thai is deepest; others are implemented but less recently benchmarked)
- Distributed rate limiting at the infrastructure/proxy layer

---

*→ Back to [Project Index](../README.md)*
