# SHA-SG

**Singapore venture capital templates under version control**

Standard Singapore law venture capital template agreements — shareholders' agreements, subscription agreements, and convertible instruments — structured for version control, clause-level tracking, and diffing across template revisions.

---

## Problem

Venture capital template agreements evolve. Industry bodies update them. Firms customize them. Regulatory changes force clause amendments. But in legal practice, version tracking means:

- A folder of Word files named `SHA_v3_final_FINAL_revised2.docx`
- No reliable way to see what changed between the 2023 and 2025 versions of a standard template
- No clause-level granularity — you can diff two Word files, but the output is unreadable noise when the document is 40 pages of dense legal text
- Firm-specific customizations are maintained informally, in partners' heads or scattered across email chains

Software engineering solved this problem decades ago with version control. This project applies the same approach to legal templates.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│             SOURCE TEMPLATES                        │
│                                                     │
│  Standard form SHA · Subscription Agreements ·      │
│  Constitution · CARE (Conv. Agreement Re Equity)    │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│          STRUCTURAL DECOMPOSITION                   │
│                                                     │
│  Full agreement → Clause-level breakdown:           │
│  Definitions · Conditions Precedent · Warranties ·  │
│  Transfer Restrictions · Board Composition ·        │
│  Anti-Dilution · Drag/Tag · Governing Law           │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│           VERSION-CONTROLLED STORAGE                │
│                                                     │
│  Git-tracked clause files · Metadata per clause ·   │
│  Amendment history · Cross-template references      │
└─────────────────────────────────────────────────────┘

```
---

*→ Back to [Project Index](../README.md)*
