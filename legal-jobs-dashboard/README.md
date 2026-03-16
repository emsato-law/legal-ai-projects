# Legal Jobs Dashboard

**Self-updating market intelligence for AI + legal roles**

An automated pipeline that monitors, aggregates, and analyzes job postings at the intersection of artificial intelligence and legal practice. Provides a continuously updated dashboard with employer analysis, role taxonomy, compensation signals, and market trend tracking — with minimal manual intervention.

---

## Problem

The AI + legal job market is fragmented across dozens of platforms, poorly categorized, and moving fast. A role titled "Legal AI Engineer" at one company is "Legal Innovation Manager" at another and "AI Policy Counsel" at a third. Searching manually means:

- Checking multiple job boards daily
- Missing roles buried under generic titles
- No historical data to track which employers are scaling AI teams
- No structured way to compare what different markets actually want

This system automates the monitoring and adds the analytical layer that job boards don't provide.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│                DATA COLLECTION                       │
│                                                      │
│  Job Board Scrapers → API Integrations →             │
│  Deduplication → Raw Posting Storage                 │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│                ANALYSIS ENGINE                       │
│                                                      │
│  Role Classification → Employer Profiling →          │
│  Requirement Extraction → Compensation Analysis →    │
│  Trend Detection                                     │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│                 DASHBOARD                            │
│                                                      │
│  Live Listings · Employer Profiles · Market Trends   │
│  · Role Taxonomy · Alert System                      │
└─────────────────────────────────────────────────────┘
```

---

## Key Components

### 1. Data Collection Pipeline

Scheduled scrapers and RSS integrations pull job postings from multiple sources. The deduplication engine identifies the same role posted across platforms and consolidates them into a single canonical record.

### 2. Analysis Engine

Raw postings are processed through an LLM-powered analysis layer:

- **Role classification.** Maps varied job titles to a normalized taxonomy (e.g., "Legal AI Engineer," "AI-Enabled Legal Ops," "Legal Innovation Counsel," "AI Policy & Governance").
- **Employer profiling.** Tracks which organizations are hiring, how frequently, and at what seniority level — revealing which firms and companies are actually building AI legal teams vs. making one-off hires.
- **Requirement extraction.** Structures the skill and experience requirements into comparable fields: technical skills, legal qualifications, jurisdiction requirements, seniority level.
- **Compensation signal detection.** Extracts salary ranges, equity mentions, and compensation benchmarks where disclosed.

### 3. Dashboard

A web-based interface providing:

- **Live listings** with normalized metadata and direct links to source postings
- **Employer profiles** showing hiring patterns over time
- **Market trend views** — what skills are in rising demand, which geographies are growing, how role definitions are evolving
- **Highlighting** for new posts or trends

---

## Design Decisions

**Why build this rather than use existing job aggregators?**
Because the AI + legal intersection is a niche that aggregators handle poorly. LinkedIn returns noise. Indeed miscategorizes. This system applies domain-specific classification that only works because it's built by someone who understands both sides of the "AI + legal" search.

**Why LLM-powered analysis rather than keyword matching?**
Because the same role is described in wildly different ways. Keyword matching would miss "Generative AI Counsel" when searching for "Legal AI." LLM classification understands semantic equivalence across varied job description language.

---

## Tech Stack

- Python (scrapers, analysis pipeline)
- LLM API calls for classification and extraction
- Scheduled execution via cron automation
- Web-based dashboard (HTML/CSS/JavaScript)

---

*→ Back to [Project Index](../README.md)*
