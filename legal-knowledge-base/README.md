# Legal Knowledge Base

**Files-first ingestion system for legal RAG**

A CLI-driven knowledge base system that transforms primary legal regulations, secondary sources, templates, and practice notes into structured, validated, searchable, and version-controlled Markdown — designed as the foundation layer for retrieval-augmented generation. Currently focused on Thai and U.S. legal materials with multi-jurisdiction support.

**Status:** Ingestion pipeline, validation, and SQLite search index operational. RAG retrieval layer in active development.

---

## Problem

Legal knowledge management is stuck between two bad options:

1. **Traditional document management** (folders, filenames, manual tagging). Lawyers can find documents they know exist but can't surface relevant passages across a large corpus. Provenance is lost. Versioning is informal. Nobody knows whether a PDF is the current version, a draft, or superseded.

2. **General-purpose RAG systems** (drop everything into a vector database, hope for the best). These treat a Ministry Notification the same as a dated internal memo. They chunk by token count rather than legal structure. They retrieve by semantic similarity without understanding that a 2025 amendment supersedes a 2019 provision.

Legal RAG needs a *legal-native ingestion layer* — one that understands document types, hierarchical structure, temporal relationships, jurisdictional scope, and bilingual instrument pairs before anything gets embedded. This system builds that layer.

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
│           CLASSIFICATION & REVIEW                   │
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
│           CANONICAL KB (kb/)                        │
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
└─────────────────────────────────────────────────────┘
```

---

## Repository Structure

```
legal-templates/
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
- **Legacy folders** are backlog reservoirs — move one item at a time into `sources/incoming/` for processing; archive each legacy folder once exhausted

---

## How It Works

### The Files-First Principle

The system is built around a strict rule: **`kb/` is the source of truth.** Everything in `build/` is derived and regenerable. The canonical legal text lives in version-controlled Markdown with structured YAML front matter — not in a database, not in a vector store, not in PDFs.

This means every change to the knowledge base is a git commit. Every version of every legal instrument has a traceable history. Diffs between versions are readable. And the entire corpus can be rebuilt from the Markdown files alone.

### One-at-a-Time Ingestion

The system processes exactly one item at a time through `sources/incoming/`. This is a deliberate constraint:

1. Place one source item into `sources/incoming/` — either a family folder (for primary law) or a single file (for notes, references, templates).
2. Run `kb process`. The system generates a starter `kb.yaml` with auto-detected metadata and prefilled fields where canonical matches exist.
3. **Human reviews the `kb.yaml`**, corrects any errors, and sets `confirmed: true`. This is not optional — the system will not ingest unconfirmed items.
4. Run `kb process` again. On success, the item moves to `sources/archive/`, the canonical file appears in `kb/`, a permanent audit record is written to `kb/_audit/`, and the search index is updated.

The one-item constraint prevents batch errors from cascading. Legal metadata requires human judgment — automated guesses about authority, instrument type, and publication date are starters, not answers.

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
| Embedding generation | 🔧 Planned |
| RAG retrieval layer | 🔧 Planned |
| Semantic query interface | 📋 Planned |

---

## Tech Stack

- Python 3.10+ (CLI application installed via `pip install -e .`)
- Custom CLI (`kb process`, `kb validate`, `kb build-index`, `kb search`)
- SQLite for full-text search indexing
- YAML front matter for structured metadata
- JSON Schema for editor autocomplete and validation
- Markdown as canonical document format
- Git for version control (the knowledge base *is* the git history)
- GitLab CI for automated tests, validation, and build smoke checks

---

*→ Back to [Project Index](../README.md)*
