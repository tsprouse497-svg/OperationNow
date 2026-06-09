# Poker Training Bot Codex Plan

Decision-reviewed scaffold plan for a fresh, offline-first, deterministic NLHE
training bot repo.

Decision review date: 2026-06-08

## 1. Intent

Build a deterministic NLHE poker training bot that can be developed by Codex
agents with minimal human coordination. The owner should be able to review phase
completion through committed reports, audit packets, and plain-language checks,
without needing to understand the codebase.

The v1 product is a CLI/report-driven offline training system. It should support
deterministic bot-vs-bot simulation, replayable hand histories, artifact-backed
preflop strategy, conservative postflop fallback, profile comparison reports,
and small normalized hand-history sample comparison.

Runtime poker decisions must not rely on LLM reasoning.

## 2. Non-Goals For V1

- No PokerNow automation.
- No browser/platform observation.
- No UI package or training UI.
- No real-time live-game advisory product surface.
- No large 30M-hand ingestion requirement.
- No runtime solver calls.
- No postflop solver-backed strategy requirement.
- No heuristic guessing for missing preflop chart spots.
- No human-maintained task routing.

Deferred v2+ work belongs in `docs/ROADMAP.md` and `backlog.yml`, not in v1
package surfaces.

## 3. Fresh Repo Decision

Create a fresh repo at:

```text
C:\OperationNow\poker-training-bot
```

The repo name is:

```text
poker-training-bot
```

Phase 0 should initialize local git, verify the scaffold, and create a local
commit. GitHub publishing happens only after the local scaffold verifies cleanly.

Git is the source of truth. This means committed repo files, local history, and
later remote branches/PRs are authoritative. It does not require publishing a
broken initial scaffold before local verification passes.

## 4. V1 Phase Order

Lock v1 to this offline-only sequence:

```text
Phase 00: Scaffold, contracts, tooling, verifier, reports, coordinator workflow
Phase 01: 2-9 player NLHE core engine with synthetic golden hands
Phase 02: Normalized hand-history schema and deterministic replay
Phase 03: Strategy contract and deterministic decision audit shape
Phase 04: Preflop artifact/chart contract, importer, and fail-closed lookup
Phase 05: Full-table preflop strategy from committed artifacts/charts
Phase 06: Conservative postflop fallback for simulation continuity
Phase 07: Offline simulator and bot/profile comparison reports
Phase 08: Tiny normalized sample ingestion and player tendency comparison
Phase 09: Quality, drift, backlog, and phase-gate hardening
```

Phase 0 must create phase contracts for all phases immediately. Later agents
must not invent acceptance tests after implementation unless they first run a
separate explicit contract-update task.

## 5. Package Layout

Use one conventional Python package under `src/`:

```text
src/poker_training_bot/
  __init__.py
  poker_core/
  hand_history/
  strategy/
  solver_artifacts/
  simulator/
  profiles/
  data_pipeline/
```

Each planned subpackage may have `__init__.py` and `README.md` in Phase 0.
Avoid deeper placeholder API files until the relevant phase starts.

Do not create v1 package folders for:

- `pokernow_adapter`
- `training_ui`

Mention those only as deferred roadmap/backlog items.

## 6. Tooling

Use `uv` and commit `uv.lock`.

Target the local Python version only for v1. Do not claim broad Python version
support until the project needs distribution.

Phase 0 includes real tooling immediately:

- `pyproject.toml`
- `pytest`
- `ruff`
- import smoke tests
- Windows and shell verification wrappers

Verification entry points:

```text
scripts/verify.ps1
scripts/verify.sh
scripts/run_verify.py
```

Both shell wrappers call the same Python verifier.

CI should exist from day one:

```text
.github/workflows/verify.yml
```

CI runs the same `scripts/verify.sh` command as local verification.

## 7. Repo Control Plane

Use these committed repo control files:

```text
CURRENT_TASK.yml        # agent-updated current assignment and task mode
phase_status.yml        # machine source of truth for phase progress
backlog.yml             # machine source of truth for deferred/proposed work
STATUS.md               # generated human snapshot
docs/PHASE_LEDGER.md    # generated human phase table
docs/BACKLOG.md         # generated human backlog
docs/phase_contracts/   # stable acceptance/test contracts
docs/exec_plans/        # coordinator plans and handoff state
```

`STATUS.md`, `docs/PHASE_LEDGER.md`, and `docs/BACKLOG.md` are generated and
committed. Verification fails if they are stale.

`CURRENT_TASK.yml` is visible, committed, and intentionally updated by agents.
It is not generated. It is the current assignment control file.

## 8. Current Task Rules

`CURRENT_TASK.yml` enforces one official active task at a time for v1.

It should include:

```yaml
task_id: PHASE-00-SCAFFOLD
active_phase: "00"
task_mode: implementation
approved_scope:
  - AGENTS.md
  - CURRENT_TASK.yml
  - phase_status.yml
  - backlog.yml
  - docs/**
  - scripts/**
  - src/**
  - tests/**
  - reports/**
forbidden_scope:
  - data/raw/**
  - data/processed/**
required_task_commands:
  - check_scope
  - check_contracts
  - check_generated_status
scope_change_log: []
```

Rules:

- `approved_scope` is strict by default.
- Agents may expand scope in the same task only by updating
  `CURRENT_TASK.yml` and appending a reason to `scope_change_log`.
- `forbidden_scope` wins over `approved_scope`.
- Contract changes mixed with implementation changes are blocked unless
  `task_mode` is an explicit contract-update mode.
- `CURRENT_TASK.yml` stays phase/package-level. Internal slices live in the
  active ExecPlan.

## 9. Coordinator Workflow

The repo should replace copy-pasted session handoffs with a coordinator flow.

When the user says "start Phase N" or "start package X", Codex should:

1. Read `AGENTS.md`.
2. Read `CURRENT_TASK.yml`, `phase_status.yml`, and the active phase contract.
3. Enter coordinator mode.
4. Create or update an active ExecPlan.
5. Break the phase/package into internal slices.
6. Delegate to subagents where available.
7. Update the ExecPlan after each meaningful slice.
8. Run verification and required reports.
9. Stop only for a true blocker, prohibited-scope violation, or completed
   phase/package gate ready for human vetting.

Required scaffold files:

```text
docs/COORDINATOR_WORKFLOW.md
docs/exec_plans/TEMPLATE.md
docs/exec_plans/active/.gitkeep
docs/exec_plans/completed/.gitkeep
```

Active ExecPlans are committed living documents and must include a
`Next Agent Bootstrap` section.

## 10. Phase Contracts

Phase contracts live under:

```text
docs/phase_contracts/
```

Each contract uses YAML frontmatter plus Markdown.

Required frontmatter fields:

```yaml
phase_id: "01"
title: "2-9 Player NLHE Core Engine"
status: future
depends_on:
  - "00"
required_gate_commands:
  - pytest_poker_core
  - generate_phase_01_replay_report
required_reports:
  - reports/active/latest_replay_report.txt
required_phase_audit: reports/phase_audits/PHASE_01_CORE_ENGINE.md
```

Required Markdown sections:

- Scope
- Non-goals
- Acceptance criteria
- Required reports
- Required command IDs
- Human vetting packet requirements
- Forbidden shortcuts
- Regression expectations

Contract changes require a separate explicit contract-update task before
implementation changes.

## 11. Verification Model

Use named command IDs, not arbitrary shell commands in YAML.

`scripts/run_verify.py` maps command IDs to known repo commands. Agents may add
new command IDs when needed, but only within approved scope and with tests for
the command registry.

Verification writes:

```text
reports/active/verify_results.json
reports/active/latest_verify.txt
```

`verify_results.json` is machine-readable. `latest_verify.txt` is
human-readable.

Command results are generated by the wrapper, not manually assembled by agents.

Command skip policy:

- Non-gate task commands may be skipped only with a required reason.
- Phase gate commands cannot be skipped at phase completion.

## 12. Reports And Audit Packets

Split reports into active generated reports and archived phase audit packets:

```text
reports/
  active/
    verify_results.json
    latest_verify.txt
    latest_replay_report.txt
    latest_strategy_query_report.txt
    latest_decision_audit.jsonl
  phase_audits/
    PHASE_00_SCAFFOLD.md
    PHASE_01_CORE_ENGINE.md
    logs/
      PHASE_01_CORE_ENGINE/
```

Active generated reports are committed only when they are part of the current
active phase's acceptance gate.

Phase audit packets are stable historical gate evidence. `scripts/verify.*`
checks completed audit packet existence and structure, but does not require old
audit packet contents to be regenerated unless that phase is being re-audited.

Phase completion is blocked unless the required phase audit packet is committed.

Each phase contract defines its own human vetting packet contents. Packets must
include plain-language pass/fail checks for a non-coding reviewer.

Audit packets should summarize command results and point to structured
verification files/logs. Do not dump long raw logs into the packet.

Detailed gate logs are committed only for active phase gate runs, with size
limits.

## 13. Common Definition Of Done

Create:

```text
docs/DEFINITION_OF_DONE.md
```

Every completed phase/package must satisfy:

- phase contract acceptance criteria met
- required gate command IDs passed
- required active reports fresh
- phase audit packet committed
- gate logs committed where required
- `phase_status.yml` updated
- generated `STATUS.md`, `docs/PHASE_LEDGER.md`, and `docs/BACKLOG.md` current
- active ExecPlan outcome/retrospective filled in
- no forbidden scope changes
- backlog updated for deferred work
- independent agent review completed where available

Independent review focuses on contract, reports, tests, and audit packet.
Reviewers inspect code only if evidence is missing, suspicious, or failing. If
an independent agent is unavailable, the phase packet must clearly say only
self-review was performed.

## 14. Command Policy

There is no human command approval workflow inside the repo. Codex agents are
expected to run project commands autonomously.

Use command categories for agent self-governance:

- Routine: tests, linters, report generation, status generation, local
  inspection.
- High-impact: dependency changes, large data processing, contract updates,
  scope expansion, branch/PR/merge operations. Agents may run these but must
  leave an auditable trail.
- Prohibited for v1: commands or code paths that implement platform automation,
  large data ingestion into git, destructive history rewrites outside explicit
  maintenance tasks, or v1-forbidden product scope.

## 15. File Size Limits

File-size limits fail verification from day one.

Recommended Phase 0 limits:

```text
AGENTS.md                           target <=120 lines, hard fail >150
STATUS.md                           generated, hard fail >120 lines
docs/*.md                           hard fail >500 lines, except ExecPlans
docs/phase_contracts/*.md           hard fail >300 lines each
docs/exec_plans/active/*.md         hard fail >800 lines
src/**/*.py                         hard fail >500 lines each
tests/**/*.py                       hard fail >700 lines each
reports/active/*.txt                hard fail >300 KB
reports/active/*.json               hard fail >1 MB
reports/phase_audits/*.md           hard fail >500 lines
reports/phase_audits/logs/**/*.log  hard fail >500 KB per log
data/samples/**                     hard fail >5 MB total committed sample data
```

If a file must exceed a limit, split it or create an explicit harness-update
task.

## 16. Testing Rules

Add anti-over-mocking guidance to:

```text
AGENTS.md
docs/TESTING_LADDER.md
docs/WORKFLOW_RULES.md
```

Rules:

- Prefer real fixtures, golden hands, replay checks, schema validation, CLI
  reports, and deterministic simulations.
- Mocks are allowed only at hard external boundaries.
- Do not mock `poker_core`, strategy legality validation, replay, or report
  generation just to pass tests.
- Phase contracts define minimum real evidence for each phase.

Do not add a Phase 0 mock-detection checker.

## 17. Hand History And Data

Use one canonical normalized hand-history schema.

Supported formats:

- `.json` for single-hand golden fixtures
- `.jsonl` for multi-hand sample datasets

Both use the same per-hand schema.

Phase 0 includes:

```text
docs/HAND_HISTORY_SCHEMA.md
docs/phase_contracts/PHASE_02_HAND_HISTORY.md
```

Phase 0 also includes 2-3 tiny synthetic example fixtures clearly marked as
scaffold examples, not proof of engine correctness.

Phase 1 golden hands start synthetic only. Sanitized real/public examples come
later in the data/replay comparison phase.

V1 data scope:

- tiny normalized samples only
- no full 30M-hand ingestion requirement
- full large-dataset comparison deferred until parser/replay reliability is
  proven
- committed sample data capped by file-size limits

## 18. Strategy And Artifacts

Preflop strategy is artifact-backed charts generated/imported offline and
consumed deterministically at runtime.

Phase 0 includes:

```text
docs/PREFLOP_ARTIFACT_CONTRACT.md
docs/phase_contracts/PHASE_04_PREFLOP_ARTIFACTS.md
```

The contract must define:

- chart/artifact schema
- source metadata
- supported table sizes and stack depths
- deterministic action selection
- missing spot behavior
- audit fields
- forbidden shortcuts

Missing preflop chart spots abstain/fail closed in v1. No heuristic guessing for
missing preflop chart coverage.

V1 includes conservative deterministic postflop fallback only after preflop and
simulator foundations exist. The fallback is safe/simple for simulation
continuity first, not sold as strong strategy.

## 19. Simulator And Comparison

Engine supports 2-9 players from day one.

Simulator gates ramp in layers:

```text
2-player -> 6-max -> 9-max
```

V1 comparison starts with bot/profile reports and tiny normalized sample
comparison. Human/public hand comparison is allowed only through the normalized
sample path first.

Profile tags stay as placeholders until baseline strategy and simulator reports
prove behavior:

- nit
- TAG
- LAG
- calling station
- loose passive
- maniac

## 20. Git And Publishing Workflow

Create:

```text
docs/GIT_WORKFLOW.md
```

Rules:

- local commits are source of truth until GitHub remote exists
- after remote exists, use branch/PR workflow
- verify before commit and before push
- phase gates need committed audit packets
- no history rewrites unless explicitly part of a maintenance task
- agents may create/push branches and PRs autonomously once a remote exists
- agents may merge phase/package-gate PRs after verification and independent
  review pass
- agents may merge ordinary slice PRs if task verification passes and work stays
  within the active ExecPlan
- phase status advances only after phase/package gate packet passes

## 21. Phase 0 Scaffold Deliverable

Phase 0 should create the fresh repo, implement the scaffold/control-plane, run
verification, generate the Phase 00 audit packet, and commit the verified local
baseline.

Required files include at least:

```text
AGENTS.md
README.md
STATUS.md
CURRENT_TASK.yml
phase_status.yml
backlog.yml
pyproject.toml
uv.lock
.gitignore
.github/workflows/verify.yml
docs/ARCHITECTURE.md
docs/WORKFLOW_RULES.md
docs/COORDINATOR_WORKFLOW.md
docs/DEFINITION_OF_DONE.md
docs/TESTING_LADDER.md
docs/HAND_HISTORY_SCHEMA.md
docs/PREFLOP_ARTIFACT_CONTRACT.md
docs/GIT_WORKFLOW.md
docs/ROADMAP.md
docs/PHASE_LEDGER.md
docs/BACKLOG.md
docs/phase_contracts/*.md
docs/exec_plans/TEMPLATE.md
scripts/run_verify.py
scripts/verify.ps1
scripts/verify.sh
scripts/generate_status.py
scripts/generate_phase_ledger.py
scripts/generate_backlog.py
scripts/check_scope.py
scripts/check_contracts.py
scripts/check_file_sizes.py
src/poker_training_bot/**
tests/**
reports/active/**
reports/phase_audits/PHASE_00_SCAFFOLD.md
```

Phase 0 acceptance:

- `uv` environment works
- import smoke tests pass
- verifier runs from PowerShell and shell wrapper
- generated human docs are current
- phase contracts exist for all v1 phases
- phase contract structure validates
- scope checker works
- file-size checker works
- committed reports required by Phase 00 are fresh
- Phase 00 audit packet exists and has the required plain-language checklist
- git repo has a verified local scaffold commit

