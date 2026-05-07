---
name: implementation-overview-plan-design
description: Design and refine an implementation overview plan document from a spec, existing code, and local implementation-doc conventions. Use when Codex needs to create or revise an overview-plan style implementation document before detailed step plans, especially when the goal is to reduce back-and-forth by locking end-to-end inputs/outputs, artifact paths, implementation structure, ownership boundaries, and unresolved design decisions early.
---

# Implementation Overview Plan Design

Use this skill when the task is to turn a spec into an implementation-facing overview plan such as `overview-plan.md`, `total-plan.md`, or an equivalent top-level implementation plan document.

This skill is specifically for the stage before detailed step plans.

## Core Rule

The overview plan must be decision-complete at the architecture and contract level.

A later implementer should not need to decide:

- what the top-level inputs are
- what the top-level outputs are
- where artifacts go
- which subsystem owns which responsibility
- whether an existing entrypoint should be reused or a new one should be added
- what high-risk ambiguities still exist

Do not write detailed step plans until the overview plan is stable.

## Primary Goal

Reduce future back-and-forth by forcing the high-impact decisions to surface early.

This means the overview plan should prefer:

- explicit contracts
- explicit ownership boundaries
- explicit artifact path rules
- explicit defaults
- explicit exclusions

over vague prose or premature implementation detail.

## Workflow

### 1. Ground In Existing Conventions

Before writing the plan:

1. Read the source spec.
2. Read at least one existing implementation-plan document in the repo.
3. Inspect the current code entrypoints, orchestration layer, report layer, and config layer that are likely extension points.

Match local document style where possible.

Do not invent a new doc style if the repo already has one.

### 2. Freeze The Plan Level

Confirm that this document is an overview plan, not a step-by-step implementation plan.

By default, the overview plan should contain:

- summary
- implementation scope
- end-to-end contract
- implementation structure and responsibilities
- major phase overview
- core behavioral rules
- hparam/config contract
- validation/test plan
- assumptions

By default, it should not contain:

- per-file edit instructions
- per-test-case enumerations for each phase
- migration scripts
- function-by-function patch order

If the user wants detailed step plans too, write the overview plan first and treat it as canonical.

If the overview plan changes later, the detailed plans must be regenerated from it.

### 3. Lock The End-to-End Contract

The overview plan must explicitly define:

- execution input format
- config location convention
- required config fields
- source-of-truth data sources
- public execution entrypoint
- output artifacts
- artifact directory layout
- default output path behavior

Always separate:

- user-provided inputs
- internally resolved inputs
- generated outputs

If the feature is run from a config file, prefer defining one config contract rather than a loose CLI flag list.

### 4. Lock The Ownership Model

The overview plan must name the major modules and state each module's single primary responsibility.

When proposing implementation structure:

- keep one main responsibility per file/module
- split modules when one file would own multiple unrelated semantics
- separate orchestration from calculation
- separate aggregation from rendering
- separate resolve/path helpers from business logic
- separate raw-data statistics from transformed-data statistics when semantics differ

Do not write “helper” or “utils” style ownership unless the responsibility is genuinely narrow and shared.

### 5. Force High-Value Decisions Early

Before finalizing the overview plan, actively look for decisions that commonly create ping-pong later.

Especially check:

- Should this reuse an existing entrypoint or create a new one?
- Is the execution target split-based or id-list-based?
- What is the exact config contract?
- Which values are optional vs required?
- Which defaults are path-derived vs config-derived?
- Are report-only fields different from execution fields?
- What is the exact artifact root path?
- Which data sources feed which subsystem?
- What is the execution unit for iterative or batched work?
- How does active work leave the processing set?
- Which null/default semantics are locked already?

If a high-impact ambiguity remains, surface it explicitly before writing the final plan.

### 6. Keep The Plan At The Right Resolution

The overview plan should be implementation-facing but not patch-level.

Good:

- module boundaries
- config contract
- artifact contract
- high-level phase outputs
- subsystem responsibilities

Too detailed for this stage:

- exact internal helper names for trivial logic
- exhaustive unit test matrices per phase
- line-by-line migration order
- speculative abstractions not required by the spec

## Required Sections

Unless the repo has a stronger convention, include these sections:

### Summary

State:

- what feature is being added
- what it intentionally does not change
- whether this is overview-plan only or includes detailed steps

### Implementation Scope

Include:

- what is included
- what is excluded

This section is valuable because it prevents future scope creep in step plans.

### End-to-End Contract

Must define:

- input config contract
- internally resolved inputs
- outputs
- artifact path rules

### Implementation Structure And Responsibilities

Must define:

- major module paths
- one-line responsibility per module
- why ownership is separated that way when non-obvious

### Phase Overview

Must define:

- major phases
- phase purpose
- phase inputs
- phase outputs

This section is the bridge between overview planning and future detailed step plans.

### Core Behavioral Rules

Include only behavior that the later implementer must not reinterpret.

Examples:

- aggregation rule
- invalid transitions
- null handling
- artifact naming contract
- batching semantics

### HParam Or Config Contract

List the config keys whose shape or semantics matter.

Prefer:

- field name
- type/allowed values
- default if fixed
- what it controls

### Test Plan

At overview stage, define validation axes, not exhaustive scenario lists.

Good examples:

- config/resolve helper unit tests
- artifact contract tests
- simulation engine unit tests
- aggregation tests
- report smoke tests
- CLI smoke tests

### Assumptions

Record the defaults chosen because they prevent future re-litigation.

## Anti-Ping-Pong Checklist

Before finalizing, check whether the plan clearly answers all of these:

- What file creates the config schema?
- How is the train config found?
- How is the output root found?
- What exactly does the user provide?
- What exactly is resolved internally?
- Where do artifacts go by default?
- What is the execution unit for iterative or batched processing?
- How does active work shrink or terminate over time?
- Which data source feeds each major subsystem?
- Is there a separate report payload stage?
- Are report builders inside the feature namespace or shared?
- Is the main public entrypoint new or reused?
- Does this document explicitly say that detailed step plans come later?

If any answer is missing, the overview plan is not ready.

## Decision Heuristics

When several reasonable options exist, prefer:

- config-file driven execution over many ad hoc CLI flags
- checkpoint-derived defaults over duplicated config paths
- explicit artifact roots over implicit relative paths
- id-list execution targets over split selectors when the feature is per-entity evaluation
- one responsibility per module over convenience grouping
- explicit optional metadata fields over overloaded primary fields
- canonical overview plan first, detailed steps second

## Output Pattern

When generating an overview plan, prefer this sequence:

1. Read the source spec and local doc convention.
2. Identify likely code extension points.
3. Lock the overview-vs-detail boundary.
4. Define end-to-end contract.
5. Define structure and responsibilities.
6. Define phase overview.
7. Define key rules and config contract.
8. Define validation axes.

If the user asks for review rather than creation, critique the overview plan using the anti-ping-pong checklist above.

## Final Check

Before considering the document complete, ask:

`Can a different engineer write the detailed step plans without reopening the same architecture questions?`

If the answer is no, the overview plan is not ready.
