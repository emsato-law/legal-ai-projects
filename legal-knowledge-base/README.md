# Legal Knowledge Base (LKB)

**Files-first canonical ingestion and review system for legal knowledge**

A files-first canonical knowledge base system that transforms primary legal regulations, secondary sources, templates, and practice notes into structured, validated, searchable, and version-controlled Markdown. The Legal Knowledge Base (LKB) is the canonical legal layer of the broader system and is currently focused on Thai and U.S. legal materials with multi-jurisdiction support.

**Status:** Reviewed source-understanding packets, approved Source Units, canonical ingest, SQLite search index, UI-assisted review, and downstream Legal Wiki draft-packet preparation are operational. RAG retrieval layer in active development.

The Legal Wiki (LW) is a separate synthesis layer built on top of LKB citations. It may summarize, compare, and organize the LKB, but it is not controlling authority.

---

## UI Screenshots

<div align="center">
  <img src="./screenshots/splash.png" alt="AI-Assisted KB Ingest" width="700">
  <br><em>AI-Assisted KB Ingest — upload a single note/reference/template or stage a primary-law family draft</em>
  <br><sub>As of 26 March 2026</sub>
</div>

<p>&nbsp;</p>

<div align="center">
  <img src="./screenshots/suggested_meta.png" alt="Review Draft screen with AI-suggested metadata" width="700">
  <br><em>Review Draft — inspect model usage, review proposed metadata, refresh suggestions, and confirm ingest</em>
  <br><sub>As of 26 March 2026</sub>
</div>

---

## Problem

Legal knowledge management is stuck between two bad options:

1. **Traditional document management** (folders, filenames, manual tagging). Lawyers can find documents they know exist but can't surface relevant passages across a large corpus. Provenance is lost. Versioning is informal. Nobody knows whether a PDF is the current version, a draft, or superseded.

2. **General-purpose RAG systems** (drop everything into a vector database, hope for the best). These treat a Ministry Notification the same as a dated internal memo. They chunk by token count rather than legal structure. They retrieve by semantic similarity without understanding that a 2025 amendment supersedes a 2019 provision.

Legal RAG needs a *legal-native ingestion layer* — one that understands document types, hierarchical structure, temporal relationships, jurisdictional scope, and bilingual instrument pairs before anything gets embedded. The LKB builds that layer.

The [Translation Pipeline](../translation-pipeline/) is the designed upstream source for multilingual legal material. It produces structure-corrected, translated legal documents in Markdown — the format the LKB expects. The goal is for translated output to flow directly into ingestion without manual reformatting, after which the LKB can serve as the authoritative citation layer for the LW.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│              SOURCE MATERIALS                       │
│                                                     │
│  sources/incoming/    ← one active item at a time   │
│  sources/archive/     ← processed items             │
│  TH-Legal/, TH-ICO-Portal/, etc. ← legacy backlog   │
└──────────────────────┬──────────────────────────────┘
                       │  kb process
                       ▼
┌─────────────────────────────────────────────────────┐
│             LKB CLASSIFICATION & REVIEW             │
│                                                     │
│  Auto-generate starter kb.yaml:                     │
│  Document type · Jurisdiction · Authority ·         │
│  Language · Temporal metadata · Provenance          │
│                                                     │
│       ↓  ← HUMAN REVIEWS, CORRECTS, CONFIRMS        │
│                                                     │
└──────────────────────┬──────────────────────────────┘
                       │  kb process (confirmed: true)
                       ▼
┌─────────────────────────────────────────────────────┐
│             LKB CANONICAL STORE (kb/)               │
│                                                     │
│  kb/primary_law/     ← versioned legal instruments  │
│  kb/notes/           ← commentary & analysis        │
│  kb/reference/       ← supporting materials         │
│  kb/templates/       ← reusable working documents   │
│  kb/_audit/          ← permanent ingest audit trail │
└──────────────────────┬──────────────────────────────┘
                       │  kb validate → kb build-index
                       ▼
┌─────────────────────────────────────────────────────┐
│           DERIVED ARTIFACTS (build/)                │
│                                                     │
│  SQLite search index · Manifest · Reports           │
│  (always regenerable from kb/)                      │
└──────────────────────┬──────────────────────────────┘
                       │  authoritative citations
                       ▼
┌─────────────────────────────────────────────────────┐
│             LEGAL WIKI (LW)                         │
│                                                     │
│  Topics · Issues · Concepts · Processes             │
│  Entities · Questions                               │
│  (source-backed synthesis, not authority)           │
└─────────────────────────────────────────────────────┘
```

---

## Repository Structure

```
LKB/
├── .agents/                ← AI workflow instructions for building and maintaining the KB
├── .gitlab-ci.yml          ← CI pipeline: tests, validation, build smoke checks
├── pyproject.toml          ← Python packaging + `kb` console-script entrypoint
│
├── kb/                     ← CANONICAL — the source of truth
│   ├── primary_law/        ← Versioned legal instruments
│   ├── notes/              ← Commentary & analysis
│   ├── reference/          ← Supporting materials by category
│   ├── templates/          ← Reusable working documents by category
│   └── _audit/             ← Permanent timestamped ingest audit records
│
├── sources/
│   ├── incoming/           ← Single active inbox item for `kb process`
│   └── archive/            ← Processed source items (done queue)
│
├── schema/                 ← Metadata reference, JSON schemas, editor schemas
├── tools/                  ← CLI commands, ingestion, validation, indexing logic
├── tests/                  ← Unit and integration regression tests
├── docs/                   ← Human-facing documentation
│
├── build/                  ← DERIVED — disposable, gitignored, always regenerable
│   ├── index.sqlite
│   └── reports/
│
│   Legacy backlog reservoirs (to ingest gradually):
├── TH-Legal/
├── TH-ICO-Portal/
├── TH-SEC.OR.TH/
├── compliance/
├── sectors/
└── annotated-pdfs/
```

The mental model:

- **`kb/`** is canonical — everything else serves it
- **`sources/incoming/`** is the active queue — one item at a time
- **`sources/archive/`** is the done queue — processed items with their reviewed `kb.yaml`
- **`build/`** is disposable — regenerate anytime with `kb build-index`
- **The LKB is authoritative** — the LW consumes its citations but does not replace it
- **Legacy folders** are backlog reservoirs — move one item at a time into `sources/incoming/` for processing; archive each legacy folder once exhausted

---

## How It Works

### The Files-First Principle

The system is built around a strict rule: **`kb/` is the source of truth.** Everything in `build/` is derived and regenerable. The canonical legal text lives in version-controlled Markdown with structured YAML front matter — not in a database, not in a vector store, not in PDFs.

This means every change to the knowledge base is a git commit. Every version of every legal instrument has a traceable history. Diffs between versions are readable. And the entire corpus can be rebuilt from the Markdown files alone.

### LKB and LW Boundary

The LKB and the LW serve different roles:

- The **LKB** stores controlling legal text, provenance, version history, and authoritative citations.
- The **LW** summarizes, compares, and organizes the LKB into living synthesis pages.
- If the LW conflicts with the LKB, the LKB wins until the synthesis is corrected.
- The LW may quote short excerpts when necessary, but it should not duplicate full legal text.

### One-at-a-Time Ingestion

The system processes exactly one item at a time through `sources/incoming/`. This is a deliberate constraint:

1. Place one source item into `sources/incoming/` — either a family folder (for primary law) or a single file (for notes, references, templates).
2. Run `kb process`. The system generates a starter `kb.yaml` with auto-detected metadata and prefilled fields where canonical matches exist.
3. **Human reviews the `kb.yaml`**, corrects any errors, and sets `confirmed: true`. This is not optional — the system will not ingest unconfirmed items.
4. Run `kb process` again. On success, the item moves to `sources/archive/`, the canonical file appears in `kb/`, a permanent audit record is written to `kb/_audit/`, and the search index is updated.

The one-item constraint prevents batch errors from cascading. Legal metadata requires human judgment — automated guesses about authority, instrument type, and publication date are starters, not answers.

### AI-Assisted Review UI

The LKB now has a browser-based review layer on top of the same canonical backend. It does not replace the `kb process` model; it gives the reviewer a more structured way to stage, inspect, confirm, and hand off an item.

Current UI behavior:

- **Primary law:** blocker-first review, reevaluation, reviewed source-understanding and Source Unit approval, immediate canonical ingest after explicit confirmation, and then optional Legal Wiki review.
- **Notes, references, and templates:** lightweight file or pasted-text intake, AI-assisted metadata review, canonical ingest, and optional Legal Wiki review.
- **Post-ingest Legal Wiki review:** staged topic overview, placement review, draft-packet review, and explicit approve/reject decisions.
- **Update proposals:** existing Legal Wiki targets can be proposed for update rather than silently ignored.
- **Auditability:** execution logs, ingest reports, reviewed Source Units, and downstream review packets keep the handoff reviewable even when the UI performs multiple behind-the-scenes steps.

For primary law, the review layer now stages a reusable source-understanding pass around canonical ingest. The system can assemble a packet with source inventory, Page Index structure, bilingual alignment, line-numbered source text, and core metadata; AI can draft Source Units for reviewer approval, and those approved units can later anchor downstream Legal Wiki draft packets without making the Legal Wiki authoritative.

The key boundary remains unchanged: canonical ingest completes in the LKB first. The Legal Wiki step uses saved ingest outputs and indexed rows as source context; it does not rerun ingest or make the wiki authoritative.

### Four Content Categories

| Category | What it is | Inbox shape | Key metadata |
|---|---|---|---|
| **Primary law** | Statutes, regulations, decrees, notifications | Family folder with working text + optional archival PDF | `work_id`, `version_id`, `jurisdiction`, `authority`, `language`, `status`, full temporal chain |
| **Note** | Commentary, analysis, deal notes, policy observations | Single top-level file | `date_published`, `tags`, optional `note_kind` (update, analysis, deal, proposal, etc.) |
| **Reference** | Supporting materials, guides, secondary sources | Single top-level file | `category`, `language`, `tags`, optional provenance |
| **Template** | Reusable forms, checklists, clause libraries | Single top-level file | `category`, `language`, `tags`, optional provenance |

Primary law carries heavy legal metadata — versioned, provenance-driven, bilingual-aware, tied to a formal issuing authority. Notes, references, and templates are lightweight by design.

### Bilingual Instrument Handling

ASEAN legal practice regularly requires working with instruments in multiple languages — a Thai statute alongside its official English translation, or a Khmer regulation with an unofficial English version. The system handles this natively:

- Thai and English variants of the same instrument live in the same family folder
- Each language version gets its own canonical file with language-tagged filenames (e.g., `instrument.th.md`, `instrument.en.md`)
- When heading labels differ between languages for the same numbered provision, `anchor_overrides` maps keep fragment IDs aligned
- Translation provenance is tracked in metadata

### Validation & Search Index

Two separate verification layers:

- **`kb validate`** checks front matter completeness, metadata values against an authoritative schema (`metadata-reference.yaml`), path conventions, and cross-references. Validation is semantic — it catches problems that a search index build would silently accept.

- **`kb build-index`** generates a SQLite full-text search index supporting queries by title, citation, heading, and full text across languages. Indexing is incremental by default, with optional full rebuild.

```bash
kb search "SPV" --language en
kb show "<document-id>" --language en
```

### Command Surface

The core LKB workflow now spans both intake and retrieval:

- **`kb help`** — render the authoritative metadata guide and field rules
- **`kb ui`** — run the local AI-assisted ingest review UI
- **`kb build-manifest`** — inventory source material and produce review reports
- **`kb init-family`** — write starter `kb.yaml` files for incoming primary-law families
- **`kb process`** — process the single active inbox item through review, ingest, archive, and index refresh
- **`kb validate`** — run semantic validation across canonical files
- **`kb build-index`** — build or rebuild the SQLite search index
- **`kb eval-source-units`** — evaluate or review Source Unit generation against canonical source material
- **`kb search`** — search indexed content by keyword and metadata filters
- **`kb show`** — inspect a specific indexed document or fragment
- **`kb drafting-suggest`** — summarize repeated lightweight-review edits into drafting preference suggestions

### Provenance & Audit

Every ingested item carries provenance metadata:

- `official_url` — where the source was obtained
- `source_hash` — integrity verification of the working text
- `source_hash_note` — explanation when hashing isn't possible
- Translation source and method for non-original-language files
- Timestamped audit reports in `kb/_audit/` with category classification, source analysis, canonical metadata snapshot, validation results, and exact indexed text

---

## Design Decisions

**Why one item at a time rather than batch ingestion?**
Because legal metadata requires human judgment. Large batches frequently contain duplicates, corrected copies, summaries mixed with real instruments, Thai and English variants, and files with similar names but different legal effect. The one-item constraint forces a review decision on every item. Speed comes from automation of everything *around* that decision — auto-detection, canonical matching, prefilled metadata, automated archival — not from skipping the decision itself.

**Why Markdown as the canonical format rather than keeping PDFs as source of truth?**
PDFs are archival. They're kept as provenance siblings, but the working source must be extracted text (`.md`, `.mmd`, `.txt`). Markdown is diffable, searchable, version-controllable, and parseable. A PDF is none of these things reliably — especially for Thai legal text, where PDF extraction routinely produces untrustworthy garbled output (See translation-pipeline for the solution to this).

**Why a canonical work registry for repeated source matching?**
Because the same law appears in dozens of source collections with different filenames, different metadata, different versions, and different extraction quality. The work registry provides stable identity — when `kb process` encounters a new source file, it can match it against known works and pre-fill metadata from confirmed previous versions, dramatically reducing review time on the second, third, and fourth encounter.

**Why SQLite for the search index rather than a vector database?**
Because the current phase is structured retrieval — finding specific provisions by citation, title, or keyword. Full-text search in SQLite is fast, auditable, and doesn't require embedding infrastructure. The vector layer for semantic retrieval is the next phase, and it will sit *on top of* this structured foundation, not replace it.

---

## Current Status & Roadmap

| Phase | Status |
|---|---|
| Primary law ingestion pipeline | ✅ Operational |
| Notes, references, templates ingestion | ✅ Operational |
| Metadata validation engine | ✅ Operational |
| SQLite full-text search index | ✅ Operational |
| Batch source inventory & review reporting | ✅ Operational |
| Canonical work registry | ✅ Operational |
| Bilingual instrument handling | ✅ Operational |
| Audit trail generation | ✅ Operational |
| Editor support (VS Code schema + snippets) | ✅ Operational |
| Source-backed downstream wiki drafting | ✅ Operational |
| UI-assisted post-ingest Legal Wiki review | ✅ Operational |
| Lightweight note/reference/template wiki handoff | ✅ Operational |
| Existing wiki page update proposals | ✅ Operational |
| Automated ingestion from [Translation Pipeline](../translation-pipeline/) | 🔧 Planned |
| Embedding generation | 🔧 Planned |
| RAG retrieval layer | 🔧 Planned |
| Semantic query interface | 📋 Planned |

---

## Tech Stack

- Python 3.10+ (CLI application installed via `pip install -e .`)
- Custom CLI (`kb help`, `kb ui`, `kb build-manifest`, `kb init-family`, `kb process`, `kb validate`, `kb build-index`, `kb search`, `kb show`, `kb drafting-suggest`)
- SQLite for full-text search indexing
- YAML front matter for structured metadata
- JSON Schema for editor autocomplete and validation
- Markdown as canonical document format
- Git for version control (the knowledge base *is* the git history)
- GitLab CI for automated tests, validation, and build smoke checks

---

*→ Back to [Project Index](../README.md)*
