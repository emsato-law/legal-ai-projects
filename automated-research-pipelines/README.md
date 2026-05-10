# Automated Research Pipelines

**Scheduled AI-driven research, summarization, and distribution**

A set of autonomous research pipelines that run on scheduled agent sessions. Each run performs targeted web research, produces structured summaries, writes source-backed dashboard data, and commits the updated research store without manual intervention. The dashboard keeps the output browsable by tag and sends article-level user feedback into the next scheduled update.

<div align="center">
  <img src="./screenshots/es-dashboard.png" alt="Research dashboard with tagged updates, feedback controls, and automation status" width="900">
  <br><em>Current dashboard snapshot - source-backed research items, clickable tags, feedback controls, and automation status</em>
  <br><sub>As of 10 May 2026</sub>
</div>

<p>&nbsp;</p>

---

## Problem

Staying current across fast-moving domains — developments across the AI legal sector, regulatory developments, industry news — requires daily manual effort: scanning multiple sources, filtering noise, synthesizing what matters, and sharing it with the right audience. This is exactly the kind of repetitive, high-frequency knowledge work that benefits from automation.

These pipelines replace that daily manual scan with scheduled, AI-driven research that runs reliably, covers more ground, preserves historical items, and keeps the output easy to browse by topic. User feedback on tags, relevance, and priority becomes input for the next run instead of disappearing into a separate notes process.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│            SCHEDULED AGENT RUN                      │
│                                                     │
│  Cron / automation → Isolated research session      │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│              RESEARCH PHASE                         │
│                                                     │
│  Web search → Source filtering →                    │
│  Content Extraction → Domain-Specific Focus         │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│            SYNTHESIS AND STRUCTURING                │
│                                                     │
│  LLM analysis → Structured summary generation →     │
│  Source Attribution → Quality Formatting            │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│              DASHBOARD OUTPUT                       │
│                                                     │
│  Historical JSON stores · Markdown content ·        │
│  Git commit/push · Dashboard status · Tag views     │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│              USER REVIEW LOOP                       │
│                                                     │
│  Article feedback → Feedback inbox →                │
│  Next scheduled research run                        │
└─────────────────────────────────────────────────────┘
```

---

## Pipelines

### 1. Daily Briefing Modules

Scheduled jobs research specific domains each morning and update dashboard-backed data stores.

Current modules:

- **Thai law and regulatory updates** — Royal Gazette and regulator-focused monitoring.
- **AI updates** — model, product, agent/tool, enterprise, safety, infrastructure, and policy developments.
- **General news** — Thailand/ASEAN, global macro, markets, geopolitics, supply chains, technology, and policy context.

**Flow:**
1. Scheduled automation starts the run.
2. The agent searches current sources for each module.
3. Items are filtered, summarized, tagged, and linked back to source URLs.
4. New items merge into existing historical item lists instead of replacing prior entries.
5. Dashboard data and content files are committed and pushed.

**Output:** Dashboard-ready JSON and Markdown content with source-backed summaries, stable item IDs, tags, dates, and impact notes. Every item exposes clickable tags for filtered historical views.

### 2. AI Legal Sector Dashboard

A daily research pipeline that tracks company developments, funding rounds, hiring signals, workflow/product changes, law firm adoption, in-house legal AI, and market structure across selected AI legal companies.

**Schedule:** 7:30 AM daily (Asia/Bangkok)

**Flow:**
1. Cron triggers at 7:30 AM
2. Spawns an isolated agent session
3. Agent searches for the latest AI legal sector news, focusing on tracked companies and exact company tags
4. New items merge into the dashboard history store
5. Company profile pages show tracked company metadata and related updates
6. Changes are committed and pushed
7. Run status is reflected in the dashboard automation status module

**Output:** Updated dashboard data, company profile pages, and research content with source-backed updates.

### 3. Feedback-Aware Updates

The dashboard includes article-level feedback. Feedback is saved locally, read by the next scheduled run, and used to adjust tagging, relevance, and future prioritization.

Supported feedback classes:

- wrong tag
- missing tag
- more like this
- less like this
- not relevant
- high priority

The next run treats that feedback as instruction-level input:

- **Wrong tag / missing tag** - reconsider the article's taxonomy.
- **More like this / less like this** - tune future selection.
- **Not relevant / high priority** - update relevance and escalation rules.

Processed feedback is archived outside the committed dashboard content.

### 4. Tag-Based Review

Tags are first-class navigation, not just labels. Each item tag links to a newest-first historical view for that topic, and module metadata exposes the canonical tag set for each research area.

Tag views support:

- cross-module review of recurring topics;
- fast audit of why an item was classified under a topic;
- company-specific AI Legal views using exact company tags;
- date-range review across 15 days, 30 days, 6 months, 1 year, or all history.

---

## Design Decisions

**Why isolated sessions?**
Each pipeline runs in its own agent session, fully isolated from the main operator session. This prevents research tasks from polluting the active context window and allows clean teardown after completion. If a pipeline fails, it fails in isolation.

**Why dashboard-backed historical stores?**
Research updates are more useful when they accumulate. The dashboard keeps historical item lists, supports tag filters and date-range views, and lets recurring topics become searchable over time rather than disappearing after a single notification.

**Why a user feedback loop?**
Automated classification needs correction paths. Article-level feedback gives the user a low-friction way to flag wrong tags, missing tags, poor relevance, or high-priority patterns. The next run reads those signals before generating new output.

**Why Git as the durable record?**
Daily research output changes over time, and the history matters. Git gives every run a diffable record of what changed, when it changed, and which data/content files were updated.

**Why keep legacy process references?**
Earlier Discord-first dashboard processes are useful as process studies, but the current dashboard does not depend on those runtime codebases. They inform the workflow shape without becoming live dependencies.

---

## Dependencies

- **Scheduled agent runtime** — isolated research sessions
- **Web search** — source discovery and verification
- **LLM analysis** — summarization, classification, and synthesis
- **Git** — version-controlled research data and content
- **Dashboard layer** — historical views, module pages, tag filtering, feedback controls, and automation status

---

## Tech Stack

- Custom agent runtime (isolated session management)
- LLM-backed summarization and classification
- Web search and source extraction
- Git-based research file management
- Local dashboard data/content stores
- Scheduled automations
- Feedback inbox and archive workflow
- Tag index and filtered historical views

---

*→ Back to [Project Index](../README.md)*
