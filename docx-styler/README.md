# Docx Styler (Outline Styler)

**AI-driven paragraph styling for Word documents**

A two-phase tool that automatically assigns Word paragraph styles (Heading 1, Heading 2, Body Text, Quote) using Google Gemini. The human stays in the loop: AI predicts styles, the user reviews predictions in a spreadsheet, then the tool applies the approved styles to produce a clean, consistently formatted document.

---

## Problem

Every lawyer knows this document:

- Heading 1 is 14pt Arial Bold on page 1, then 13pt Times New Roman Bold on page 4
- Body text alternates between three different paragraph styles depending on who drafted which section
- Numbering restarts unexpectedly because someone copy-pasted from a different template
- "Normal" style means something different in every section

This happens because legal documents accumulate formatting like sediment — each drafter's Word defaults, each firm's template, each round of tracked changes leaves a layer. Manual cleanup takes hours and is error-prone. Most lawyers give up and ship inconsistent documents.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│            PHASE 1: EXTRACT + PREDICT               │
│                                                     │
│  Input DOCX                                         │
│       │                                             │
│       ▼                                             │
│  Paragraph Extraction (text, position, context)     │
│       │                                             │
│       ▼                                             │
│  Gemini Classification                              │
│  (predict style per paragraph)                      │
│       │                                             │
│       ▼                                             │
│  Excel Worksheet (worksheet_pred.xlsx)              │
│  with AI predictions for human review               │
│                                                     │
│       ↓  ← HUMAN REVIEWS & CORRECTS IN EXCEL        │
│                                                     │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│            PHASE 2: REAPPLY STYLES                  │
│                                                     │
│  Reviewed Excel + Original DOCX                     │
│       │                                             │
│       ▼                                             │
│  Style Application Engine                           │
│  (maps approved styles back to paragraphs)          │
│       │                                             │
│       ▼                                             │
│  Clean, consistently styled output DOCX             │
└─────────────────────────────────────────────────────┘
```

---

## How It Works

### Phase 1 — Extract + Predict

The tool reads the input Word file, extracts every paragraph with its content and positional context, and sends them to Google Gemini for style classification. Gemini predicts the appropriate style for each paragraph from a configurable set of allowed styles.

The predictions are written to an Excel spreadsheet (`worksheet_pred.xlsx`) with one row per paragraph and the AI's predicted style in a dedicated column. The user opens this in Excel or Google Sheets, reviews the predictions, corrects any mistakes, and saves.

This review step is the core design choice: **Gemini is good but not perfect, and styling decisions in legal documents carry consequences.** A heading misclassified as body text breaks the document's navigable outline. The spreadsheet gives the user full visibility and control before anything is written back to the document.

### Phase 2 — Reapply

The tool takes the reviewed spreadsheet and the original Word document, maps the approved style assignments back to each paragraph, and writes a new styled DOCX. The output document has clean, consistent Word styles — visible in Word's Navigation Pane and properly structured for table of contents generation.

Direct formatting (inline bold, italic, font overrides) is cleared so that style definitions take effect consistently. The result is a document where formatting comes from styles, not from manual overrides — the way Word was designed to work, and the way almost no legal document actually works.

---

## Configurability

The style vocabulary is user-defined. The default set:

```python
ALLOWED_STYLES = ["Heading 1", "Heading 2", "Body Text", "Quote"]
```

This can be extended to include legal-specific styles (e.g., Heading 3, Recital, Schedule Heading, Numbered List) depending on the document type and firm template requirements. Gemini classifies into whatever style set is provided.

---

## Design Decisions

**Why a two-phase workflow with a spreadsheet review step?**
Because fully automated restyling is a trust problem. Lawyers need to see what the AI decided *before* it touches the document. The spreadsheet is a familiar, low-friction review interface — no new tool to learn, no black box. It also creates an audit trail of what was changed and why.

**Why Google Gemini rather than a rules-based classifier?**
Because the same formatting can mean different things in context. A bold, 12pt line might be a Heading 3 in one section and an emphasized body paragraph in another. Rule-based approaches match formatting patterns; Gemini reads the content and understands that "ARTICLE IV — REPRESENTATIONS AND WARRANTIES" is a heading regardless of what font it's in.

**Why clear direct formatting on output?**
Because direct formatting is the root cause of the problem. A document where Heading 1 is *defined* as 14pt Arial Bold but *overridden* paragraph-by-paragraph with inline formatting is worse than unstyled — it looks right until someone tries to update the template, generate a table of contents, or export to another format. Clean styles mean predictable behavior.

---

## Tech Stack

- Python (python-docx, pandas, openpyxl)
- Google Gemini API for paragraph classification
- Excel/Sheets as the human review interface
- Local execution — no server deployment needed

---

*→ Back to [Project Index](../README.md)*