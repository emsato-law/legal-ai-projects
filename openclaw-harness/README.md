# OpenClaw Harness

**LLM-to-OS bridge for local task execution**

A system that connects large language models to the local operating system, enabling AI-driven execution of real-world tasks. Routes between models based on task complexity, manages a library of reusable mini-applications, and provides operational visibility through a unified dashboard.

---

## Background

This repository contains a customized OpenClaw deployment with an extended LobsterBoard control surface. The implementation adds a number of operational modules to the standard stack, including scheduled automation, mini-app extensions, and expanded telemetry.

LobsterBoard acts as the operational interface for the system. In this deployment it manages:
* mini-app modules that extend agent capabilities
* multiple cron-driven workflows
* execution metrics and operational statistics
* task monitoring and activity logs

These components sit above the OpenClaw control plane, which coordinates task intake, model invocation, and tool routing. Execution occurs through the standard OpenClaw agent loop, where the model plans actions, invokes tools, observes results, and iterates until completion.

The runtime environment exposes the system capabilities available to the agent, including filesystem access, shell execution, application control, clipboard interaction, and network operations.

---

## System Architecture

```
┌──────────────────────────────────────────────────────┐
│                  LOBSTERBOARD                        │
│          Dashboard / Control Interface               │
│                                                      │
│  ┌────────────┐ ┌────────────┐ ┌────────────────┐    │
│  │ Mini-App   │ │ Cron Job   │ │  Analytics &   │    │
│  │ Manager    │ │ Scheduler  │ │  Statistics    │    │
│  └──────┬─────┘ └──────┬─────┘ └───────┬────────┘    │
└─────────┼──────────────┼───────────────┼─────────────┘
          │              │               │
          ▼              ▼               ▼
┌──────────────────────────────────────────────────────┐
│                OPENCLAW CONTROL PLANE                │
│                                                      │
│   Task Intake · Session Handling · Tool Routing      │
│   Model Invocation · Workflow Coordination           │
│                                                      │
└───────────────┬──────────────────────────────────────┘
                │
                ▼
┌──────────────────────────────────────────────────────┐
│               AGENT REASONING LOOP                   │
│                                                      │
│   Prompt → LLM → Plan → Tool Call                    │
│        ↑                     ↓                       │
│        └──────Observe Result / Iterate───────────────┘
└───────────────┬──────────────────────────────────────┘
                │
                ▼
┌──────────────────────────────────────────────────────┐
│              LOCAL EXECUTION ENVIRONMENT             │
│                                                      │
│  File Operations · Shell Commands · Application      │
│  Control · Clipboard · Network Requests              │
└──────────────────────────────────────────────────────┘
```

The LobsterBoard layer is modular in this implementation. Mini-apps provide domain-specific tooling, cron jobs trigger scheduled workflows, and analytics modules expose runtime performance and activity metrics. These extensions sit alongside the standard OpenClaw control plane without modifying the core agent loop or execution runtime.

---

*→ Back to [Project Index](../README.md)*
