---
name: monitoring-pr-ai-reviews
description: Use when a CodePeek task is already implemented, a GitHub PR exists or must be opened against Miss-you/CodePeek, and follow-up work is still needed because Copilot or other AI review comments may arrive after the initial PR push.
---

# Monitoring PR AI Reviews

## Overview

Close the loop after implementation: open the PR, keep AI review visible on a schedule, and handle each suggestion with CodePeek-aware judgment instead of blind agreement.

**Core principle:** AI review comments are inputs to evaluate, not orders. Fix real bugs, races, contract gaps, missing verification, and accessibility / packaging regressions. Reject speculative public APIs, premature cross-agent abstractions, or expansions of scope.

**REQUIRED SUB-SKILLS:** `receiving-code-review`, `verification-before-completion`

## When to Use

Use this skill when:

- the task code is already implemented and verified locally
- a PR exists or must be created now against `Miss-you/CodePeek`
- Copilot or other AI review comments may still arrive after the first push
- the remaining work is PR refresh, comment triage, fixes, replies, and thread resolution

Do not use this skill for:

- initial feature design (use `compatibility-first-planning`)
- implementing the original task from scratch (use `delivering-task-end-to-end`)
- human review that changes product direction and needs a new design decision

## The Flow

1. Confirm the task is really ready for PR.
   - Run fresh verification (`npm run lint && npm run typecheck && npm run test`, plus `npm run test:e2e` if UI / IPC was touched).
   - Open a non-draft PR. Link the OpenSpec change id and the task id from `<topic>-design-task.md` in the description.
2. Keep periodic monitoring on.
   - Primary tooling (always available): `gh pr view <num> --comments` and `gh api repos/Miss-you/CodePeek/pulls/<num>/comments` for snapshots.
   - Optional scheduled monitor (only if it exists in this repo): `.github/workflows/pr-ai-review-monitor.yml` and/or a script under `scripts/`. Treat this as future tooling — do not assume it is in place; create an issue or a `todo.md` entry if you wish it existed.
3. Read review threads, not flat comments.
   - Keep thread ids, file anchors, outdated state, and resolution state.
   - Use the GraphQL form when you need thread state:
     `gh api graphql -f query='{repository(owner:"Miss-you",name:"CodePeek"){pullRequest(number:<num>){reviewThreads(first:100){nodes{id isResolved isOutdated comments(first:5){nodes{path body author{login}}}}}}}}'`
4. Evaluate each AI suggestion against CodePeek principles.
   - preserve user-visible compatibility first (panel UX, IPC schema, persistence schema, i18n keys)
   - prefer the smallest fix that closes the real issue
   - keep internal seams internal unless a real external caller needs them
   - reject "future agent X" abstractions without a concrete current need
5. Implement only justified changes.
   - Tie each code change to a review thread.
   - Run the narrowest proving test first (the suite or file the change touches), then broader gates.
6. Refresh the PR.
   - push
   - reply with fix or rejection rationale on the relevant thread
   - resolve threads only after the code or rationale is visible on the PR
7. Re-scan until no unresolved AI review remains, or only explicit user-owned decisions remain.

## Quick Reference

| Need | Action |
| --- | --- |
| Open or refresh PR | create a non-draft PR from the verified task branch in the worktree |
| List comments on a PR | `gh api repos/Miss-you/CodePeek/pulls/<num>/comments` |
| Inspect review threads with state | GraphQL query above (resolved / outdated / file path / body) |
| Resolve a thread | `gh api graphql -f query='mutation($id:ID!){resolveReviewThread(input:{threadId:$id}){thread{isResolved id}}}' -F id=<thread-id>` |
| Re-check after push | repeat the comment / thread query and confirm zero unresolved AI review threads |

## Evaluation Rules

Accept when the comment identifies:

- a real bug, race, or broken IPC / persistence contract
- a startup, hot-reload, or shutdown failure for actual users
- missing verification (e.g. an e2e path that the test strategy claimed to cover)
- accessibility, keyboard, or focus regressions in the panel
- i18n breakage (missing key, wrong fallback)
- packaging / signing / auto-update regressions
- design or task-board drift versus the approved `<topic>-design.md` or OpenSpec change

Reject when the comment mainly asks for:

- new exported APIs without current callers
- cross-agent abstractions added "for later"
- public hooks whose only consumer is same-package test code
- refactors that widen task scope after the task is already complete
- 1:1 mirroring of CodePal where the surface no longer applies to CodePeek

When uncertain, write the rationale on the thread and ask for explicit user approval before changing scope.

## Common Mistakes

- resolving threads before the fix or rationale is visible in the PR
- treating Copilot suggestions as mandatory
- expanding the public API or adapter interface to satisfy test-only feedback
- re-running only one package's tests after a repo-wide behavioral change
- forgetting that new AI review may arrive after the first fix push
- silently accepting a "make this match CodePal" suggestion without checking whether the surface still applies
