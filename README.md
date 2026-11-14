# Hidden State Machine Debugger

A standalone coding-agent skill for discovering explicit and implicit state machines, investigating broken transitions, and planning evidence-based fixes. Read-only auditing is the default.

This repository contains Markdown instructions and reference templates. It has no executable debugger, runtime dependencies, or automated test suite. A compatible coding agent performs the investigation using its available tools and repository access.

## Installation

From the root of the project you want to audit:

```sh
git clone https://github.com/Yashuuuu02/hidden-state-machine-debugger.git .agents/skills/hidden-state-machine-debugger
```

Use a destination that does not already contain an installation. If your agent expects another skills directory, such as `.claude/skills`, substitute that path. Reload the agent if needed; discovery depends on the agent you use.

For a ZIP installation, copy the extracted folder containing `SKILL.md` and `references/` into your agent's skills directory. Keep these files together.

## Quick start

Ask your coding agent:

> Use the `hidden-state-machine-debugger` skill in `AUDIT_ONLY` mode. Discover explicit and implicit state machines across this repository. Map events, guards, invariants, and cross-machine interactions. Investigate potential failures and report confirmed defects separately from unproven risks, with source locations and P0–P3 priorities. Do not modify code or data. Return the report in chat.

For a focused review, name a feature or lifecycle, such as session refresh, offline synchronization, or upload recovery after restart.

## Operating modes

| Mode | Purpose | Changes |
|---|---|---|
| `AUDIT_ONLY` (default) | Map machines and investigate candidate defects. | No tracked file changes or external side effects. A report file is allowed only when requested. |
| `FIX_PLAN` | Design an evidence-based fix and regression test. | Does not apply either. |
| `FIX_APPROVED` | Fix explicitly authorized findings. | Minimal changes within the approved scope, followed by verification. |

To request a plan:

> Use `hidden-state-machine-debugger` in `FIX_PLAN` mode for BUG-001. Explain the root cause, proposed correction, regression test, and validation steps without applying changes.

To approve a specific fix after reviewing the findings:

> Use `hidden-state-machine-debugger` in `FIX_APPROVED` mode to fix BUG-001 only. Establish the root cause, compare a working pattern, validate one hypothesis, create a failing regression test where feasible, apply a minimal fix, and verify it. Report the commands executed and their results.

An audit request does not authorize fixes. Audit mode prohibits database writes, migrations, resets, deployments, credential changes, dependency installation, production calls, and tests that mutate persistent data or infrastructure.

## How it works

1. **Map the system.** Inspect architecture, features, data lifecycles, and the boundaries between UI, services, APIs, storage, and background work.
2. **Discover machines.** Identify state encoded in enums, coupled booleans, nullable fields, timestamps, queues, tokens, caches, and persisted records.
3. **Model transitions.** Record ownership, events, guards, side effects, invariants, concurrent actors, persistence, and recovery behavior.
4. **Challenge transitions.** Consider duplicate, late, reordered, or lost events; retries; cancellation; partial writes; concurrent editors; account changes; offline recovery; and process restarts.
5. **Investigate candidates.** Follow root-cause investigation, working-pattern comparison, and validation of a single falsifiable hypothesis.
6. **Report or fix.** Present evidence and a remediation plan, or apply minimal corrections and verification when explicitly authorized.

The workflow examines interactions between machines, such as logout while an offline queue is committing. Before fixing, it requires a root cause and a focused failing regression test whenever feasible. After three failed fix attempts, it calls for discussing the architectural state model before a fourth attempt.

## Expected report

- Inspected architecture, coverage, exclusions, and tests executed or unavailable.
- Machine inventory with source references, states, transitions, invariants, and unknowns.
- Confirmed bugs with evidence, reproduction status, event sequence, impact, and root cause.
- Unproven risks kept separate, with concrete validation plans.
- Cross-machine interactions and prioritized remediation with regression checks.
- Change log describing actual changes and verification results.

Severity and confidence are independent:

| Severity | Meaning |
|---|---|
| `P0` | Evidenced active or near-certain destructive data loss, security breach, or hard outage. |
| `P1` | High-impact corruption, authorization failure, a stuck core flow, or frequent severe breakage. |
| `P2` | Recoverable functional bug, intermittent failure, or limited inconsistency. |
| `P3` | Low-impact robustness, coverage, or maintainability concern. |

Confidence labels are `CONFIRMED`, `HIGH_CONFIDENCE`, `SUSPECTED`, and `NEEDS_RUNTIME_EVIDENCE`. Suspicious state combinations alone do not confirm a defect. Findings must distinguish observations from inference and disclose missing runtime evidence.

## Files

| File | Purpose |
|---|---|
| [SKILL.md](SKILL.md) | Skill metadata, guardrails, discovery procedure, debugging phases, and deliverables. |
| [references/audit-checklist.md](references/audit-checklist.md) | Coverage checklist for state, timing, persistence, synchronization, and evidence. |
| [references/report-template.md](references/report-template.md) | Starting template for inventories, findings, and remediation plans. |

## Limitations

Coverage depends on accessible source, logs, tests, and runtime evidence. A repository-wide audit is a systematic coverage attempt, not a guarantee that every state or defect has been found. Timing-dependent and distributed failures may require runtime validation. These instructions guide an agent's behavior; they do not enforce a technical sandbox.

## Provenance

The investigation methodology is adapted in part from the four-phase process in [obra/superpowers: systematic-debugging](https://github.com/obra/superpowers/tree/main/skills/systematic-debugging), alongside the Hidden State Machine Debugger audit workflow. The skill is self-contained; installing the original skill or its companion skills is not required.
