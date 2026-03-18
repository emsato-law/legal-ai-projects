# Automated Research Pipelines

**Scheduled AI-driven research, summarization, and distribution**

A set of autonomous research pipelines that run on cron schedules via OpenClaw. Each pipeline spawns an isolated agent session, performs targeted web research, produces structured summaries using LLM analysis, and distributes results to designated channels — with no manual intervention.

<div align="center">
  <img src="./screenshots/lobsterboard.png" alt="LobsterBoard Dashboard" width="700">
  <br><em>LobsterBoard — Cron schedules, activity log, system monitoring, and live sessions</em>
  <br><sub>As of 18 March 2026</sub>
</div>

---

## Problem

Staying current across fast-moving domains — AI legal tech hiring, regulatory developments, industry news — requires daily manual effort: scanning multiple sources, filtering noise, synthesizing what matters, and sharing it with the right audience. This is exactly the kind of repetitive, high-frequency knowledge work that benefits from automation.

These pipelines replace that daily manual scan with scheduled, AI-driven research that runs reliably, covers more ground, and delivers consistent output.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│               CRON TRIGGER                           │
│                                                      │
│  LobsterBoard Scheduler → Spawn Isolated Session     │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│              RESEARCH PHASE                          │
│                                                      │
│  Web Search (Brave) → Source Filtering →              │
│  Content Extraction → Domain-Specific Focus           │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│            AI SUMMARIZATION                          │
│                                                      │
│  MiniMax AI → Structured Summary Generation →         │
│  Source Attribution → Quality Formatting               │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────┐
│              DISTRIBUTION                            │
│                                                      │
│  Discord Channels · Git Commit & Push ·               │
│  LobsterBoard Sync · Main Session Report              │
└─────────────────────────────────────────────────────┘
```

---

## Pipelines

### 1. Daily News Summaries

Scheduled cronjobs that research specific news topics each morning, produce concise summaries, and post them to designated Discord channels.

**Flow:**
1. Cron triggers at set times in the morning
2. Spawns an isolated agent session with MiniMax AI
3. Agent searches for news on a specific topic or domain
4. MiniMax generates a structured summary with source links
5. Formatted summary is posted to the target Discord channel
6. Summary is reported back to the main session
7. Isolated session is deleted after completion

**Output:** Formatted Discord posts with summaries and links to original sources.

### 2. Legal AI Jobs Dashboard

A daily research pipeline that tracks hiring activity, funding rounds, and market movements across key AI legal tech companies — then updates a structured research file and publishes to the LobsterBoard dashboard.

**Schedule:** 7:30 AM daily (Asia/Bangkok)

**Flow:**
1. Cron triggers at 7:30 AM
2. Spawns an isolated agent session
3. Agent searches for the latest AI legal tech news, focusing on target companies (Harvey AI, Legora, Luminance, Spellbook, GC AI, Zania, SCOREalytics, August)
4. Agent updates `memory/research/ai-legal-tech-companies.md`:
   - Company valuations and funding table
   - Job opportunities section
   - In-demand roles section
   - Latest updates section
5. Changes are committed and pushed to GitLab
6. Summary is posted to `#ai-legal-jobs` on Discord
7. Summary is reported back to the main session

**Output:** Updated research file → synced to LobsterBoard at 6 AM → Discord notification.

---

## Design Decisions

**Why isolated sessions?**
Each pipeline runs in its own agent session, fully isolated from the main OpenClaw conversation. This prevents research tasks from polluting the main session's context window and allows clean teardown after completion. If a pipeline fails, it fails in isolation.

**Why MiniMax for summarization?**
MiniMax 2.5 Pro handles the routine summarization work — it's fast, cost-effective, and produces consistent output for structured research tasks. Complex reasoning tasks are routed elsewhere per the OpenClaw model routing strategy.

**Why Discord as the distribution channel?**
Discord provides immediate, low-friction delivery to the right audience with rich formatting support. Webhook integration is simple and reliable. The research files in Git serve as the durable record; Discord serves as the notification layer.

---

## Dependencies

- **MiniMax API** — AI summarization and research synthesis
- **Brave Search** — Web search for research phase
- **GitLab** — Version-controlled research file storage
- **Discord webhooks** — Channel-based result distribution
- **LobsterBoard** — Cron scheduling and dashboard sync

---

## Tech Stack

- OpenClaw agent runtime (isolated session management)
- MiniMax 2.5 Pro (summarization and research)
- Brave Search API (web research)
- Discord webhook integration
- Git-based research file management
- Cron scheduling via LobsterBoard

---

*→ Back to [Project Index](../README.md)*
