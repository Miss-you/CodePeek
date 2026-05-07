---
name: compatibility-first-planning
description: Use when CodePeek work starts from a broad goal — adding a new agent integration, redesigning the dashboard, reshaping IPC or persistence — and the scope, contract surfaces, or core/adapter boundaries are still unclear.
---

# Compatibility-First Planning

## Overview

Turn a broad CodePeek goal into an implementation-ready design without letting agent-specific assumptions, terminology drift, or speculative future-provider abstractions leak into the runtime core.

**Core principle:** Freeze external contracts and terminology before introducing abstractions.

This skill is for design-level planning. Once the design is approved, hand off to `writing-plans` (or `openspec-propose` for a single-change scope) for execution-level planning.

## When to Use

Use this skill when:

- the request starts as "add support for X agent", "redo the dashboard", "redesign persistence", "rework IPC", or another broad goal
- user-visible behavior matters across more than one surface (panel UI, IPC, persistence, agent adapters, i18n, packaging)
- multiple agent adapters (Cursor, Claude Code, Codex, CodeBuddy, …) might be touched
- future extensibility matters but overdesign is a real risk
- naming may drift between the inspiration source (CodePal), an agent's native vocabulary, and CodePeek's domain model

Do not use this skill for:

- a small isolated bugfix or copy tweak
- a single-file change with already-clear acceptance criteria
- task-by-task execution after the design is already approved (use `delivering-task-end-to-end`)

## Required Outputs

Your planning output must contain all of these, either as separate docs or as sections in one design doc under `docs/plans/<topic>-design.md`:

1. Goal and non-goals
2. Current-system inventory
3. Compatibility contract
4. Terminology mapping
5. Core vs adapter-shell boundary
6. Phase plan
7. Rough task breakdown
8. Verification and parity plan
9. Current-system evidence references

If any of these are missing, the plan is not implementation-ready.

## CodePeek Surfaces That Need Explicit Contracts

When freezing contracts, at minimum cover the surfaces that exist (or are planned) in CodePeek:

- Agent adapter interface — how each integration reports session lifecycle, tool calls, quota
- Renderer ↔ main IPC — message names, payload shape, error semantics, async vs streaming
- Persistence schema — sessions, activity events, quota snapshots, migration policy
- Dashboard UI surfaces — panel layout modes, timeline rendering, badge/state visuals
- Configuration & integration repair — config file locations, repair scripts, permission prompts
- i18n behavior — locale detection, fallback rules, copy keys
- Packaging & launch — auto-start behavior, update channel, signing/notarization expectations

Do not leave any of these as "we'll compare later" if the change touches them.

## The Flow

### 1. Clarify the target

Define what success means in user-facing terms.

Lock these early:

- what must be preserved (existing dashboards, persisted history, configured integrations)
- what may change internally (data shape, IPC names, file layout)
- what is explicitly out of scope for v1

If "match CodePal" is requested, ask which surfaces count:

- panel layout and interaction model
- agent detection and integration logic
- persistence and history restoration
- quota / rate-limit display
- integration-repair UX
- bilingual (en / zh-CN) behavior

CodePal is *inspiration*, not a strict port. Treat parity as a checklist of surfaces, not a goal in itself.

### 2. Inventory the current system

Before proposing architecture, identify:

- real runtime behavior in CodePeek today (the repo may still be partially scaffolded — say so)
- existing agent adapters and their assumptions
- implementation accidents that should not be preserved blindly
- where CodePal solves the same problem and how it differs

Back inventory claims with evidence:

- point to concrete files, IPC channels, persistence shapes, UI components, configs
- prefer a small number of high-signal references over broad code dumps
- do not claim something is "core" or "adapter-facing" without showing where that behavior exists today

Separate:

- true core behavior (agent-neutral session/activity/quota logic)
- adapter-shell behavior (per-agent integration glue)
- future extension ideas

### 3. Freeze compatibility contracts

Write down the external contracts before abstraction work starts.

Examples:

- IPC channel names, payload schemas, error codes
- persistence file format and version field
- agent adapter capability surface (what every adapter must / may provide)
- UI accessibility, keyboard, and resize contract for the floating panel
- locale fallback chain and copy key naming

Do not leave compatibility as "we'll compare later".

### 4. Write terminology mapping

Create an explicit mapping when:

- CodePal uses a term and CodePeek wants to keep, rename, or split it
- an agent's native term ("turn", "exchange", "task", "run") needs a CodePeek-internal name
- internal and external (UI / persistence) language may intentionally differ

Example questions:

- What is the internal name for an agent message exchange? Is it `Activity`, `Turn`, `Event`?
- Which user-facing labels still say "session" for compatibility with CodePal mental models?
- Which agent-specific names must stay out of the core (e.g. Codex-specific protocol terms)?

### 5. Draw the architectural boundary

Define the smallest possible agent-neutral core.

For CodePeek, the default posture is:

- keep session aggregation, activity timeline, quota domain types, persistence layer, IPC routing, and panel rendering in the core
- keep agent detection, agent-specific protocol handling, integration-repair scripts, and per-agent UI affordances in the adapter shell
- only abstract what the current system already proves is common across at least two agents

Only abstract what existing adapters already prove is common.

### 6. Phase the work around closed loops

The earliest meaningful milestone must close a real loop end-to-end.

Prefer this shape:

1. boot the desktop app
2. load config
3. detect one agent (or use a stub source)
4. ingest sessions and activity for that one agent
5. render them in the panel
6. persist and restore on restart
7. shut down cleanly

Do not make the first phase "all adapters" or "all UI states".

### 7. Define checks before implementation

Every phase needs explicit verification:

- scope check
- boundary check
- compatibility check
- operational closed-loop check (boot → render → persist → restart)
- extension check
- maintenance check

If the plan says "we will test later", it is incomplete.

### 8. Hand off to execution planning

Once the design is approved:

- freeze the design doc at `docs/plans/<topic>-design.md`
- run `deriving-task-board-from-design` to produce `docs/plans/<topic>-design-task.md`
- for each task, use `openspec-propose` (or plain `writing-plans`) to land an executable change

## Quick Reference

| Planning artifact | Question it answers |
| --- | --- |
| Goal and non-goals | What are we actually building in v1? |
| Current-system inventory | What behavior exists in CodePeek today? |
| Compatibility contract | What must remain stable for users and existing integrations? |
| Terminology mapping | How do CodePal / agent-native terms map into CodePeek terms? |
| Architecture boundary | What belongs in core vs adapter shell? |
| Phase plan | What order closes risk earliest? |
| Rough tasks | What big chunks of work exist? |
| Verification plan | How will we prove correctness and parity? |
| Evidence references | What concrete files / channels / schemas justify the plan? |

## Default Checks

Run these checks against every major design:

- `Scope`: Is v1 solving one concrete end-to-end user path?
- `Boundary`: Is each abstraction required now, or only hypothetical?
- `Compatibility`: Is every user-facing surface explicitly covered?
- `Operational`: Can the app boot, observe sessions, render, persist, and stop cleanly?
- `Extension`: Would adding another agent later require rewriting the core?
- `Maintenance`: Is the design understandable to whoever will actually maintain it?

## Red Flags

Stop and fix the plan if you notice any of these:

- "We'll figure out compatibility as we implement."
- "Let's make the adapter interface generic now for future agents."
- "Persistence schema can keep its own runtime truth, separate from the in-memory store."
- "i18n / accessibility / packaging are just polish; defer."
- "Terminology mapping is obvious; we can skip it."
- "We'll add verification after the code exists."
- "CodePal does it this way, so we should mirror it 1:1." (without checking whether the surface still applies)

## Common Rationalizations

| Excuse | Reality |
| --- | --- |
| "We should generalize the adapter interface now so adding the next agent is easy." | Premature generalization makes the core unstable. Preserve only proven seams. |
| "Persistence and the in-memory session store should share one clean store." | That collapses two truths with different lifecycles. Keep the orchestrator as the single runtime owner; persistence is a projection. |
| "Dashboard layout can be checked after the runtime works." | UI is part of the user-facing contract and must be planned explicitly. |
| "Term mapping is clerical, not architecture." | Missing term mapping causes naming drift and the wrong abstractions. |
| "Auto-start, packaging, and updater are minor." | These are real compatibility surfaces when users already have CodePeek installed. |
| "A second observability store will make rendering cleaner." | Duplicate truth creates drift. Single source for runtime state. |

## Common Mistakes

- Jumping from a vague goal to package layout without freezing IPC and persistence contracts
- Mixing agent-specific protocol assumptions (e.g. Codex-only fields) into the core
- Letting "future agent X" support dictate v1 abstractions
- Treating CodePal as a strict port instead of an inspiration
- Phases that do not close a real operational loop
- Architecture notes without a verification plan

## Output Standard

A good planning result for CodePeek should leave the reader knowing:

- what the v1 target is
- what is intentionally deferred
- how CodePal / agent-native terms map into CodePeek terms
- what the agent-neutral core is
- what remains adapter-facing
- what order the work should happen in
- how parity and correctness will be proven

If any of those are unclear, the plan is not ready.

## Next Step

After approval:

1. Use `deriving-task-board-from-design` to produce the task board.
2. Use `delivering-task-end-to-end` to claim and execute one task.
