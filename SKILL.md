---
name: hidden-state-machine-debugger
description: Standalone, self-contained skill combining implicit/explicit state-machine discovery with systematic root-cause debugging. Use to audit whole applications, identify hidden transitions, race conditions, invalid states, sync/auth/persistence issues, reproduce defects, and (only with approval) implement minimal verified fixes. No other skills required. Read-only by default.
---

# Hidden State Machine Debugger — Unified Skill

## Purpose and key rule

This is **one self-contained skill**. Do not require, install, invoke, or defer to `systematic-debugging` or another skill. This document contains the debugging process directly, integrated with state-machine discovery. Its investigation method is adapted from the four-phase process described in obra/superpowers' systematic-debugging skill, alongside original state-machine audit procedures.

**NEVER CHANGE CODE TO FIX A BUG BEFORE ESTABLISHING ITS ROOT CAUSE.** Finding a suspicious state combination is not evidence that it occurs. Do not report conjecture as a confirmed defect.

## Operating modes and guardrails

- Default: `AUDIT_ONLY`. Read source, inspect existing logs/tests, derive state models, create a report. **No tracked file changes, DB writes, migrations, resets, deployments, credential changes, dependency installs, production calls, or external side effects.** Do not run tests that write persistent data or mutate infrastructure. Writing an audit report is allowed only if the user requests it; otherwise return report in chat.
- `FIX_PLAN`: carry out evidence-based analysis and design a fix and regression test without applying either.
- `FIX_APPROVED`: only if the user explicitly authorizes implementation, fix the named defects in scope. Prefer a test-first, minimal patch with verification. Approval for an audit is not approval for fixing.
- Never reveal secrets or customer data. Treat credentials, access controls, existing deployment configuration, database integrity, and working integrations as protected boundaries.
- Inspect the actual current repository architecture rather than assuming any previously used technology.
- Label confidence: `CONFIRMED` (reproduced/tested or unambiguous deterministic execution path), `HIGH_CONFIDENCE` (strong code evidence, reproduction absent), `SUSPECTED` (plausible unproven), `NEEDS_RUNTIME_EVIDENCE` (cannot establish behavior statically).
- Record scan exclusions and gaps. “All” means make a systematic coverage attempt, **not** promise mathematical completeness.

## Stage A — map the system and find machines

1. Read project docs, manifests, dependency injection, entry points, routing, state management, repository/service layers, API/backend, DB schema/migrations, queues, workers, storage, integrations and tests. Inventory features and data lifecycles. Ignore vendored/build/generated output in primary scans.
2. Map each actual boundary: UI -> controller/state store -> repository -> API/edge function -> database/storage -> background event -> UI. Track authority and ownership of state, whether transient, cached, persisted, remote, or distributed. Check authentication and authorization at the real enforcement point.
3. Discover **implicit machines** represented by coupled booleans, nullable fields, enums, status strings, timestamps, retry counters, revision numbers, queue entries, futures, streams, locks, session tokens and DB flags. Look for implicit combinations not described by a single enum.
4. Search especially: auth/session refresh/role changes; login/logout; app bootstrap and resume; navigation; form draft/save/submit/review; optimistic CRUD; network loading/error/retry; offline pending/sync/conflict/tombstones; uploads and file metadata; OAuth connect/refresh/revoke; jobs/webhooks; retries and idempotency; transactions; cross-account/device access; caches and invalidation.
5. For **each** discovered machine, record ID, feature, code paths and exact references, variables, possible initial/current/terminal states, events, guarded transitions, side effects, invariants, owning component, persistence boundary, concurrent actors, retry/recovery behavior, and relevant tests.
6. Model transitions with a table (from | event | guard | action | to | failure/recovery) or compact state diagram. Identify unreachable or sticky states and invalid combinations. Distinguish documented business rules from assumed ones.
7. Challenge each transition: event twice/out of order/late; auth changes mid-request; app restart/process death; offline and reconnect; timeout after successful remote commit; concurrent editors; partial DB/storage success; lost acknowledgments; duplicate retry; cancellation/disposal; cross-tenant/account data; stale cache; late callback; background job race. Examine cross-machine interactions, not only each machine in isolation.
8. When useful and safe, use an isolated/local model test or property-based test to probe invariants; never infer from a test that was not executed. Prefer static reasoning in audit-only mode when tests would mutate working state.

See `references/audit-checklist.md`. Capture scan coverage (folders/features examined; inaccessible files and unknown runtime details).

## Stage B — systematic debugging, integrated for every candidate finding

The **four phases below are mandatory, in order, before any fix**. In `AUDIT_ONLY`, conduct all feasible investigation and hypothesis validation without modifying code, and output the proposed fix rather than Phase 4's implementation.

### Phase 1 — Root-cause investigation

- Read full errors, stack traces and warnings. Record precise paths, lines, events, and environment conditions.
- Attempt consistent, safe reproduction; write exact steps, input states and timing. If not reproducible, collect further evidence and mark uncertainty; **do not guess**.
- Inspect recent changes (`git diff`, commits), dependencies, environment differences and regression timing.
- At component boundaries, trace inputs, outputs and state changes; locate **where** the invariant first breaks. Use existing logs/trace points in read-only modes. If instrumentation is necessary, propose it; add it only with explicit authorization.
- Trace bad values/state **backward** through callers, events, stale writes, caches and persisted records until the original transition, not merely the visible symptom.
- State the causal chain: triggering event -> state/guard mismatch -> illegal transition -> user-observable impact, with evidence and counterexamples.

### Phase 2 — Pattern analysis

- Find an analogous working flow in the same codebase; compare its guards, serialization, state ownership, error handling, transactions, retries, and lifecycle cleanup.
- Read any relevant reference implementation fully before borrowing a pattern.
- Explicitly list differences, assumptions, environment/config dependencies and why they could matter.
- Do not decide on a remedy until the failure mechanism is explained.

### Phase 3 — Single hypothesis, minimal validation

- Write one falsifiable hypothesis: “Because [specific condition], event [X] produces invalid state [Y], evidenced by [path/log/test].”
- Select the smallest safe experiment: focused existing test, isolated mock, local reproduction, trace comparison or state-table proof. Change **one variable at a time**, if changes are authorized.
- State observed result separately from inference. If disproven, return to Phase 1 with a fresh hypothesis; never stack speculative fixes.
- If necessary evidence cannot be collected, explicitly classify as a risk and give a concrete validation plan.

### Phase 4 — Minimum correction and verification (`FIX_APPROVED` only)

- Create a focused **failing regression test first**, whenever feasible, showing violation of the identified invariant. If it is impossible, document why and establish alternate reproduction evidence.
- Make one minimal change at the **source** of the illegal transition, not a superficial workaround. Preserve interfaces, migrations, security boundaries and unrelated integrations.
- Verify the focused test turns green; run adjacent state-transition, concurrency, integration, auth/RLS, and relevant regression suites available in the environment. Inspect the diff for unrelated edits. Report exact executed commands and outcomes; don't claim unexecuted checks passed.
- If the attempt fails, stop patching and return to Phase 1. After **three failed fix attempts**, stop and discuss whether the architectural state/ownership model is flawed before a fourth attempt.
- If the problem is genuinely external/timing-dependent after thorough investigation, propose bounded timeout/retry/idempotency/observability with an explicit evidence basis; don't fabricate a local root cause.

## Severity and confidence are independent

- `P0`: evidenced active/near-certain destructive data loss, security breach, or hard outage.
- `P1`: high-impact data corruption, auth boundary failure, stuck core flow or frequent severe breakage.
- `P2`: recoverable functional bug, intermittent failure, limited-impact inconsistency.
- `P3`: low-impact robustness, coverage or maintainability concern.

Avoid calling every hypothetical race P0/P1. Each finding must clearly state severity, confidence and reproduction status.

## Deliverable (audit or fix plan)

1. **Coverage:** architecture actually inspected, exclusions, tests executed/not executed, constraints.
2. **Machine inventory:** machine ID, owner/files, states, transition summary, invariants and unknowns. Capture **every reasonably identifiable** machine found, not just a handful of examples.
3. **Confirmed bugs (first):** identifier, severity/confidence, exact source location, expected invariant, triggering timeline, actual result/evidence, reproduction, root cause, blast radius, focused test and minimal fix proposal.
4. **Unproven risks (separate):** missing evidence and how to validate them.
5. **Cross-machine interactions** (e.g. logout while an offline queue is committing, duplicate upload callbacks, retries across app restart).
6. **Prioritized remediation plan**, one root cause per change, dependency order, regression checks, rollback guidance.
7. **Change log:** explicitly declare what was and was not changed. In approved fixing mode, list tests with actual results and unresolved issues.

Use `references/report-template.md` for machine/finding details. If the repo is large, produce a complete index and clearly mark unexamined scope.

## Invocation templates

**Audit (default):**
> Use the `hidden-state-machine-debugger` skill in `AUDIT_ONLY` mode. Discover explicit and implicit state machines across this entire repository, map their events/guards/invariants and cross-machine interactions, then systematically investigate reproducible failures. Report confirmed defects separately from hypotheses, with file evidence and P0–P3 priorities. Do not modify anything.

**Fix only approved findings:**
> Use the `hidden-state-machine-debugger` skill in `FIX_APPROVED` mode to fix findings [IDs] only. Perform root-cause investigation, working-pattern comparison, one-hypothesis validation, a failing regression test, a minimal fix and verification. Leave all unrelated code, data and integrations unchanged.

## Provenance

Unified and adapted from the earlier Hidden State Machine Debugger workflow and methodological concepts in `obra/superpowers`' `systematic-debugging` (https://github.com/obra/superpowers/tree/main/skills/systematic-debugging). This unified document is standalone; external skill installation is **not** required. It does not import the original skill files or its optional companion skills.
