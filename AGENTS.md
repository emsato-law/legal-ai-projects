# Public Editorial Guide

This repository is public. It documents the architecture, design decisions, and system flows behind a portfolio of legal AI projects whose implementation lives in private repositories.

Use this file as the source of truth for public-facing rewrites, README updates, naming normalization, and project-scope decisions.

## Purpose

- Explain what each project is solving.
- Describe the architecture, product shape, and key design decisions at a high level.
- Show enough implementation detail to make the engineering choices credible and concrete.
- Avoid disclosing confidential implementation details, internal operations, credentials, private infrastructure, client information, or sensitive workflow data.

## Public Naming Map

Use these names consistently when maintaining this public repo:

- `legal-templates` = `Legal Knowledge Base` / `LKB`
- `legal-synthesis` = `Legal Wiki` / `LW`
- `docx-styler` = `DOCX Styler`

In public-facing prose, prefer the public names `Legal Knowledge Base`, `LKB`, `Legal Wiki`, and `LW`. Do not introduce private-repo names in README copy unless there is a specific maintainer reason to do so.
Do not use `LKB`, `LW`, or similar project abbreviations on any page until that page has first introduced the full public name in the form `Legal Knowledge Base (LKB)` or `Legal Wiki (LW)`.

## Projects In Scope

The public portfolio currently includes:

- Translation Pipeline
- Legal Knowledge Base
- Legal Wiki
- Automated Research Pipelines
- Fee Proposal Generator
- DOCX Styler
- SHA-SG

## Editorial Boundary

- This repo is an architecture and design portfolio, not a source-code mirror.
- The implementation repos are private; this repo should describe systems, not expose internals.
- Public docs may describe system behavior, workflows, constraints, and major components.
- Public docs should not include hidden prompts, private repository layout beyond what is already intentionally public, secrets, internal URLs, private dashboards, or operational credentials.

## Tone And Style

- Write in a clear, confident, professional tone.
- Favor concrete engineering explanation over hype.
- Keep claims accurate, restrained, and supportable.
- Prefer architecture, workflow, and design-rationale framing over marketing language.
- Assume an informed technical reader who wants signal, not slogans.
- On each page, use acronyms only after introducing the full public name on that same page.

## Formatting Rules

- Prefer Title Case headings.
- Lead each project section with a short, plain-language description line.
- Use short bullet lists for key capabilities or design decisions.
- Keep screenshots and captions factual, with dates in the format `As of 18 March 2026`.
- Use consistent public naming throughout a document once a name is established.
- When abbreviations have not yet been defined on that page, describe relationships as: `Translation Pipeline -> Legal Knowledge Base -> Legal Wiki`.
- Keep file-level prose concise; avoid turning README files into exhaustive specifications.

## Rewrite Rules

When rewriting or expanding docs in this repo:

- Preserve the distinction between canonical legal source material and synthesis.
- Treat the LKB as the authoritative legal layer.
- Treat the LW as synthesis-only.
- State clearly that the LW depends on LKB-backed citations and does not replace controlling authority.
- Prefer public category names over private implementation names.

## Default Project Framing

Use these defaults unless a maintainer asks for a deliberate change:

- **Translation Pipeline:** legal and regulatory document translation system
- **Legal Knowledge Base:** canonical legal knowledge layer with ingestion, provenance, validation, audit trail, and search
- **Legal Wiki:** synthesis-only layer built on top of LKB citations
- **Automated Research Pipelines:** scheduled AI-driven research, summarization, and distribution
- **Fee Proposal Generator:** retired internal drafting tool
- **DOCX Styler:** retired document-formatting tool
- **SHA-SG:** version-controlled Singapore venture capital template set

## Maintenance Rule

If README content conflicts with this file, follow this file and update the README accordingly.
