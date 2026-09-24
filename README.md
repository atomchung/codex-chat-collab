# Codex + ChatGPT Collaboration

This repository publishes the `codex-chat` skill and collects a public, sanitized issue backlog for the Codex ↔ ChatGPT collaboration workflow.

## The skill

The published skill is [`skills/codex-chat/SKILL.md`](skills/codex-chat/SKILL.md). It separates planning-only requests from explicitly authorized full-cycle work. In a full cycle, Codex carries a project question to ChatGPT through the available ChatGPT UI, ChatGPT uses the GitHub Connector to inspect the selected project and create a scoped issue, Codex implements it, and ChatGPT reviews the exact pushed PR head before Codex merges an approved change.

Project work belongs in the selected target repository. Issues in this repository are reserved for defects or gaps in the `codex-chat` skill and its collaboration workflow.

## What belongs in the issue backlog

Open an issue when the skill or its use exposes a material collaboration problem: a broken or ambiguous handoff, missing context, a ChatGPT review that lacks the evidence it needs, a finding Codex cannot verify, a stale-head review, a missed risk, an unhelpful finding, repeated rework, or a missing workflow step. Record which side was involved (Codex, ChatGPT, or their handoff) and what evidence supports the observation.

A defect in the product being reviewed is out of scope by itself. Record it here only when it demonstrates a failure or gap in the Codex ↔ ChatGPT collaboration workflow.

## How to record an issue

- Search existing issues first; add a recurrence there when it is the same failure mode.
- Use the [skill feedback issue form](https://github.com/atomchung/codex-chat-collab/issues/new/choose). Separate observed facts from cause hypotheses.
- Identify the Codex behavior, ChatGPT behavior, or handoff that failed; capture expected and actual behavior, impact, corrective action, and verification status.
- Keep review effectiveness measures descriptive: accepted, rejected, duplicate, missed, and rework signals need context and evidence. Do not treat raw counts as a score.
- Update the issue when a fix is made and when it is verified.

## Public information rule

This repository is public. Do not publish holdings, trades, unpublished strategies, personal or customer data, credentials, private repository/PR/branch/commit links, internal paths, or raw screenshots and transcripts. Generalize product details and remove identifying context. Keep only the minimum sanitized evidence needed to improve the skill.
