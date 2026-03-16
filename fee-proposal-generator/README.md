# Fee Proposal Generator

**Template-driven fee proposal automation for a regional law firm**

A web application that generates structured, firm-formatted fee proposals from guided questionnaire input. Replaces a manual drafting process that typically takes 2–3 hours with a structured workflow that takes minutes — while enforcing consistency across jurisdictions, deal types, and practice groups. Deployed on the firm's internal network at DFDL Legal & Tax.

---

## Problem

Fee proposals in legal practice are simultaneously high-stakes and tedious. They must be:

- **Accurate** — scoping errors become write-offs or client disputes
- **Consistent** — firm formatting, standard terms, approved language across every office
- **Jurisdiction-aware** — a Thailand incorporation proposal has different scope, pricing, and regulatory steps than a Cambodia or Vietnam proposal
- **Fast** — clients expect proposals within hours, not days

Yet in most firms, proposals are assembled manually: open a previous proposal, find-and-replace the client name, adjust the scope section by hand, guess at the hours, format it, review it, send it. Every proposal is a one-off exercise.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│                  WEB FRONTEND                        │
│              (Tailwind CSS + JS)                     │
│                                                      │
│  Template Selection → Guided Questionnaire →         │
│  Conditional Logic → Review & Generate               │
└──────────────────────┬──────────────────────────────┘
                       │ API calls
                       ▼
┌─────────────────────────────────────────────────────┐
│                FastAPI BACKEND                        │
│                                                      │
│  GET /api/templates/list                             │
│  GET /api/questions/parts/{part_id}                  │
│  GET /api/questions/conditions                       │
│  POST /api/generate/document                         │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│             DOCUMENT ENGINE                          │
│                                                      │
│  Template Selection → Answer Injection →             │
│  Section Assembly → DOCX Merge →                     │
│  Page Number & Section Break Handling →              │
│  Final Document Output                               │
└─────────────────────────────────────────────────────┘
```

---

## How It Works

### 1. Template Selection

The user selects a jurisdiction-specific template (e.g., Thailand, Cambodia). Each template maps to its own deal type data, practice manager options, and question set. A template manifest (`template_manifest.json`) defines these mappings.

### 2. Guided Questionnaire

Approximately 60 questions organized across 7 parts, presented sequentially.

**Conditional logic** drives the flow: answers to early questions determine which later questions appear. This is managed through a dedicated conditions engine (`conditions.json`) that evaluates dependencies in real time, so the user only sees questions relevant to their specific engagement.

### 3. Document Generation

Once the questionnaire is complete, the backend:

- Selects the correct DOCX template for the jurisdiction
- Injects answers into template placeholders
- Assembles optional add-on sections (annexes, supplementary terms) based on questionnaire responses
- Merges all sections into a single document with proper section breaks
- Handles independent page numbering per section (main document vs. annexes)
- Outputs a firm-branded, print-ready DOCX

### 4. Section & Page Number Management

Legal documents have specific formatting requirements that generic document generators mishandle. This system manages:

- **Section breaks** (Next Page / Continuous) between document parts
- **Independent page numbering** per section — the main body counts separately from annexes
- **Footer formatting** preserved across merged sections
- **Template-level control** — each DOCX template defines its own section structure

---

## File Structure

```
fp_generator/
├── main.py                    ← FastAPI application entry
├── template_routes.py         ← Template selection API
├── question_routes.py         ← Question loading + conditions API
├── document_routes.py         ← Document generation API
├── merge_docx.py              ← Document + section merging engine
├── models.py
├── config.py
├── utils.py
├── tests/                     ← Unit tests
├── output/
│   └── logs/                  ← Session logs
└── static/
    ├── DocGen.js              ← Primary frontend operations
    ├── DocState.js            ← State management
    ├── styles.css
    ├── views/
    │   └── index.html
    ├── templates/             ← DOCX templates per jurisdiction
    │   ├── thailand.docx
    │   ├── template_manifest.json
    │   ├── sections/          ← Add-on document sections
    │   ├── deal_types/        ← Deal content data (JSON per jurisdiction)
    │   └── pms/               ← Practice manager data (JSON per jurisdiction)
    └── questions/
        ├── parts.json         ← 7-part question structure
        ├── conditions.json    ← Conditional display logic
        └── questions.json     ← ~60 questions with metadata
```

---

## Deployment

The application ran on DFDL's internal infrastructure:

- **Server:** Linux VM with Nginx reverse proxy
- **Runtime:** FastAPI via Uvicorn, managed by systemd
- **Access:** Internal network only (VPN required for remote access)
- **SSL:** Internal certificate (valid through June 2027)
- **Backups:** Daily automated backups at 2:00 AM (Asia/Bangkok)
- **Source control:** GitLab with deployment via git pull

This was not a prototype — it was a production internal tool with proper service management and access control.

---

## Design Decisions

**Why a custom web app rather than a Word template with mail merge?**
Because the conditional logic was the hard part. Mail merge can fill in a client name and date, but it can't show or hide entire sections based on deal type, suppress irrelevant questions based on jurisdiction, or assemble optional annexes based on engagement scope. The 60-question conditional questionnaire was what makes this useful — it encoded institutional knowledge about what a fee proposal for a given deal type in a given jurisdiction should contain.

**Why DOCX output rather than PDF?**
Because fee proposals get edited after generation. A partner reviews the draft, adjusts language, modifies pricing. DOCX output lets them work in Word as usual. The tool handles the 90% that's structural; the partner handles the 10% that requires judgment.

**Why no AI in the generation pipeline?**
AI-powered scope narrative generation was on the roadmap. The firm subsequently licensed an enterprise legal AI platform, which absorbed that use case. The fee proposal generator focused on what it does well: structured template assembly with conditional logic — a problem that doesn't need AI, just good engineering.

---

## Tech Stack

- Python (FastAPI, Uvicorn)
- python-docx for template processing and section merging
- JavaScript (frontend state management and questionnaire logic)
- Tailwind CSS
- Nginx + systemd on Linux VM
- JSON-driven configuration (questions, conditions, templates, deal types)

---

*→ Back to [Project Index](../README.md)*
