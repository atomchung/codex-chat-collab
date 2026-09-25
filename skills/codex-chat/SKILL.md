---
name: codex-chat
description: Use when users want Codex to take a project question to ChatGPT for GitHub-backed planning, and continue through issue creation, implementation, exact-head review, and merge when they explicitly request or have established authorization for the full loop.
---

# Codex + ChatGPT Project Loop

Codex owns the task from kickoff through implementation and final reporting. ChatGPT contributes project planning and an independent review through its GitHub Connector. Codex uses the available computer-use interface to carry messages between the two systems and verifies each handoff from visible evidence.

Use this skill when the user explicitly asks Codex to start or continue the Codex + ChatGPT project loop, asks Codex to carry a question through that loop, or has already established full-cycle authorization for the target project. Treat “start today’s project loop and ask ChatGPT what to work on” as a full-cycle request. A request for a recommendation alone—including “what should we work on today?” without a request to run the loop or standing authorization—is planning-only. Do not infer authority to create issues, change code, push, or merge from a project name or question alone. If the mode is unclear, return the recommendation and ask whether to proceed before the first write. If the user supplies a specific question in a full-cycle request, carry it and its relevant context into the workflow.

## Roles and boundaries

- **Codex:** clarify the target project, dispatch the user’s question, inspect the created issue, implement the accepted scope, run relevant checks, prepare a PR, verify ChatGPT’s review, revise when needed, and merge an approved exact head when the user’s request authorizes the full cycle.
- **ChatGPT:** use the GitHub Connector to inspect the selected target repository, search for duplicate or related issues, propose a bounded task, create the target-project issue, and review the exact pushed PR head.
- **GitHub Connector:** ChatGPT’s source for repository, issue, and PR state. A generated answer that merely describes repository facts is not proof that the connector was used.
- **Computer-use interface:** Codex’s transport into the ChatGPT UI. In Codex, use the available `mcp__cua_repl.js` tool and its `cua` API. A prepared prompt is not a sent prompt; a sent prompt is not a completed ChatGPT response.

This skill does not grant access or expand the user’s authorization. Use the target repository named by the user or clearly established by the current task. A planning-only request ends after ChatGPT’s recommendation; it does not create an issue, change code, push, or merge. Ask only when the repository, requested scope, or authorization for a consequential step is genuinely unclear.

## 1. Establish the project and the run mode

1. Identify the target GitHub repository (`OWNER/REPO`) from the user’s request and the active project context. Confirm the current Git remote when working in a checkout. Do not substitute `atomchung/codex-chat-collab`; that repository is only for skill-level feedback.
2. Set the run mode before dispatch. A request to start or continue the full project loop, or an established standing authorization for that target project, selects `FULL_CYCLE`; carry through issue creation, implementation, review, and merge when the repository rules and review gates pass. A request for a recommendation or plan alone selects `PLANNING_ONLY`; return ChatGPT’s answer and stop before issue creation, code changes, push, or merge. If neither mode is clear, return the recommendation and ask before the first write. Do not stop for redundant confirmation after the user explicitly selected `FULL_CYCLE`.
3. Inspect the checkout’s branch, commit, and working tree before changing files. Preserve unrelated work. Gather only the concise context ChatGPT needs; do not send credentials, private personal data, or unrelated local files.
4. Search this skill’s public issues only when the run reveals a material skill or handoff problem. Search and create product work in the target project repository.

## 2. Send the question to ChatGPT through the UI

Use the host’s available computer-use tool to operate the signed-in ChatGPT web UI. In Codex, the current route is `mcp__cua_repl.js`:

1. Call `cua.getState()` to identify the browser/profile for the intended signed-in ChatGPT account. For a normal handoff, open a **new browser tab and a new ChatGPT conversation** with `cua.createBrowserTab(...)`; do not bind or send into whichever ChatGPT conversation happens to be open. Existing personal chats are never a fallback target. Bind an existing tab only when the user provides its exact URL and explicitly designates that conversation for this machine task. If the intended account/profile is unclear, ask before sending.
2. Treat one independent Codex task as one machine-owned ChatGPT conversation. Continue later phases of that same task (such as the PR review in a full cycle) in its verified conversation URL. Give each concurrent or independent Codex task a separate conversation, and keep at most one in-flight request per conversation. Never reuse a personal conversation merely because it contains related project context.
3. Verify the new conversation is blank or visibly new before sending. If the new tab opens an existing conversation, use the visible New Chat action and read the accessibility state again. If Codex cannot establish and verify a fresh conversation, leave the handoff pending; do not fall back to an ambiguous tab.
4. Label each machine conversation `codex-<short-topic>` (for example, `codex-atlas-issue-17`). If the visible UI offers a rename action, rename it and verify the displayed title. If renaming is unavailable, put a stable `Codex session: codex-<short-topic>` marker at the start of the first message and do not claim the visible title changed. Retain the exact conversation URL in the active Codex task context/report so later phases can return to that machine-owned conversation. Do not put private URLs or project details in public skill-feedback issues.
5. Verify the intended account/session, target repository, conversation identity, and visible GitHub Connector availability before asking ChatGPT to act. Read the accessibility state, identify the current composer element, and paste the task message into it with `tab.paste(composerIndex, message, {format: "text"})`. Submit with `tab.pressKey(composerIndex, "Return")` or click the visible send control. If no accessible composer is exposed, use a fresh screenshot and visible UI coordinates. After each UI action, call `tab.getAXState()` before deciding what to do next; rediscover element indexes instead of reusing stale ones. Verify that the submitted user message appears in the intended machine conversation, then wait for and verify the matching assistant reply. Do not treat a queued message or an unrelated turn as completion.
6. Verify from the visible ChatGPT interaction that the GitHub Connector was actually invoked for repository inspection or issue creation. If the connector is unavailable, the wrong conversation is open, or the UI send/read-back cannot be verified, report the handoff as pending and provide the ready-to-send message. Do not claim a dispatch or connector action that was not observed.
7. For every critical GitHub read that supports a repository fact or consequential next step, match the visible request identity to the visible result identity before treating it as verified. At minimum, match the repository and requested resource type plus its issue/PR number or exact ref when applicable. If the result is missing, truncated before those identifiers, belongs to a different repository or resource, or is otherwise ambiguous, mark that read and any dependent fact as unverified. A Connector badge, invocation count, or neighboring response is not sufficient evidence; do not create/update issues, implement, approve a review, or merge based on an unverified critical read.

For a daily planning request, ask ChatGPT to inspect the target repository and its open issues, identify one high-value task that fits the user’s context, check for duplicates, and explain its evidence, scope, acceptance criteria, risks, and dependencies. For a user-supplied question, include it faithfully and ask ChatGPT to answer it in the context of the target repository.

For a full cycle, ask ChatGPT to create one issue in the **target project repository** after selecting a concrete task. The issue should state the problem, bounded scope, acceptance criteria, relevant evidence, and material risks. Ask for the issue URL and confirm the Connector created it in the intended repository. If a duplicate exists, reuse or update that issue instead of opening a duplicate. If ChatGPT cannot create the issue, stop before implementation and report the blocker; do not silently create a different issue in the skill-feedback repository.

## 3. Implement the target-project issue

1. Read the resulting issue from the target repository. Confirm that its title, repository, scope, and acceptance criteria match the ChatGPT plan and the user’s request. Stop if the issue is in the wrong repository, duplicates existing work, or materially expands the agreed scope.
2. Inspect the current checkout and repository instructions. Implement the issue on an appropriate branch, preserving unrelated changes. Run the relevant tests and checks for the change; report exactly what ran and what did not.
3. Push and open or update a PR when needed for ChatGPT to inspect the remote diff. Include enough context to connect the PR to the target issue. Record the PR URL and the exact head SHA after the push. Do not ask ChatGPT to review local changes it cannot see.
4. For UI or generated-content work, include sanitized rendered evidence and the corresponding candidate text in the PR or an authorized review artifact. Identify the candidate SHA for that evidence. Keep source review and rendered-surface review distinct; ChatGPT must say which evidence it actually inspected.

## 4. Request and correlate the ChatGPT review

Send the review request through the same verified ChatGPT UI path. Ask ChatGPT to use the GitHub Connector on the PR, verify the exact head SHA, inspect the diff and necessary context, and check the acceptance criteria. The response must include:

- Target repository and PR URL.
- Exact head SHA inspected.
- `APPROVE`, `REQUEST CHANGES`, or `CANNOT REVIEW`.
- For each finding: severity, file and line or other precise location, code evidence, impact, and a concrete correction.
- The number of findings returned and an overflow status: `none` or `additional findings remain`, with a concise severity summary if the response is capped.
- Which CI or rendered evidence ChatGPT actually inspected, and any material limits on the review.

Codex must match the response’s repository, PR URL, and SHA to the requested review before using its verdict. A missing or mismatched response is `CANNOT REVIEW` for that head. Never borrow a verdict from a neighboring request. Whenever the response reports additional findings, ask ChatGPT for the next bounded set for the same repository, PR, and SHA. Continue until ChatGPT explicitly reports that no additional findings remain. If overflow status is missing or unclear, treat the review as incomplete. Do not report review completion or merge while overflow is unresolved or unknown.

Codex independently checks every finding against the exact diff and relevant tests. Explain evidence for rejected findings. Fix valid findings, rerun relevant checks, push the changes, record the new SHA, and request a new review of that head. A verdict for an earlier SHA expires as soon as the PR head changes.

If the review transport or GitHub Connector is unavailable, leave review status pending and do not merge. Give the user the prepared prompt and the specific missing evidence.

## 5. Merge and report

Merge only when all of the following are true:

- The user’s request authorizes the full implementation and merge loop.
- ChatGPT returned `APPROVE` for the exact current repository, PR URL, and head SHA using the GitHub Connector.
- Review overflow is explicitly clear, no material findings remain unresolved, and all required CI checks are green.
- The target branch and repository rules permit the merge.

Immediately before merging, reread the PR head and required check state. Submit the merge with a head-SHA precondition, such as `gh pr merge <number> --match-head-commit <reviewed SHA>`, GitHub GraphQL `expectedHeadOid`, or an equivalent guarded operation. If the head changed or the precondition is unavailable or rejected, leave the PR unmerged, inspect the new head and checks, and request a new exact-head review; never retry the merge against an unreviewed SHA. If any merge condition fails, keep the PR unmerged and state what is missing. After a successful merge, verify the resulting PR state and merge commit through GitHub. Do not imply deployment or live-client validation from a successful merge.

Report the target issue and PR, the implemented scope, tests run, CI state, ChatGPT’s exact-head verdict, merge state, and any remaining deployment or user-facing validation. Keep local test evidence, GitHub checks, ChatGPT review, merge, deployment, and client validation as separate claims.

## 6. Feed material skill failures back into this project

When the run exposes a material failure in Codex’s workflow, ChatGPT’s behavior, or their handoff, search the open issues in `atomchung/codex-chat-collab` first. Add evidence to an existing issue when it is the same failure mode; otherwise create a sanitized skill-gap issue using the repository’s issue form. Keep observed facts separate from cause hypotheses and include expected behavior, actual behavior, impact, proposed skill change, and verification status.

Do not record a target product defect here unless it demonstrates a gap in this collaboration skill. This repository is public: omit private repository, PR, branch, or commit links; personal or customer data; credentials; raw screenshots; and private transcripts. Keep only the minimum evidence needed to improve the skill. Describe acceptance, rejected findings, duplicates, misses, and rework with context rather than turning raw counts into a score.

## ChatGPT message templates

### Daily project kickoff

> Run mode: `<PLANNING_ONLY or FULL_CYCLE>`. We are working in `<OWNER/REPO>`. Use the GitHub Connector to inspect the repository and its open issues. Find one high-value, in-scope task for today, check for duplicates, and explain the evidence, scope, acceptance criteria, risks, and dependencies. In `PLANNING_ONLY`, return the recommendation without creating an issue. In `FULL_CYCLE`, create or reuse the issue in this repository and return its URL. Do not claim a connector read or issue write you did not perform.

### User-supplied question

> Run mode: `<PLANNING_ONLY or FULL_CYCLE>`. The user’s question is: `<QUESTION>`. Relevant constraints: `<CONTEXT>`. Use the GitHub Connector to inspect `<OWNER/REPO>` where needed and answer in that project’s context. In `PLANNING_ONLY`, return the answer and a short plan without creating an issue. In `FULL_CYCLE`, turn the agreed bounded task into a new or existing issue in this repository and return its URL. State whether the Connector was used.

### Exact-head PR review

> Use the GitHub Connector to review `<PR URL>` in `<OWNER/REPO>`. First verify that its current head is exactly `<SHA>`. Goal and acceptance criteria: `<SUMMARY>`. Inspect the diff and necessary context; inspect the linked rendered evidence when applicable. Return `APPROVE`, `REQUEST CHANGES`, or `CANNOT REVIEW` for that exact head, with precise evidence and actionable findings. Return at most three findings per response, state the total or whether additional findings remain, and summarize the severity of any overflow. Say which CI and rendered evidence you inspected. Do not modify the repository or dispatch another Codex task.
