---
name: implementation-detailed-plan-design
description: Design and refine detailed implementation step-plan documents from an approved overview plan, existing code, and local implementation-doc conventions. Use when Claude needs to create or revise step-by-step implementation plan documents such as `01-*.md`, `02-*.md`, and similar files after the overview plan is stable, especially when the goal is to reduce back-and-forth by locking per-step inputs/outputs, file ownership, function contracts, schema fields, and test scenarios before coding starts.
---

# Implementation Detailed Plan Design

Use this skill when the task is to turn an approved overview plan into detailed step-plan documents.

This skill is specifically for the stage after the overview plan is stable and before code implementation starts.

## Core Rule

Each detailed step plan must be implementation-complete at the function-contract level.

A later implementer should not need to decide:

- which files belong to the step
- which functions exist in each file
- which functions are public or private
- what each function is responsible for
- what each function takes as input
- what each function returns
- which models or schemas exist
- what fields each model or schema contains
- which test scenarios prove the step is correct

Do not write detailed step plans until the overview plan is stable.

If the overview plan changes later, delete and rewrite all detailed step plans from the revised overview plan.

## Primary Goal

Reduce future back-and-forth by forcing per-step implementation decisions to surface early.

This means each detailed plan should prefer:

- explicit file ownership
- explicit function contracts
- explicit schema/model fields
- explicit input/output examples
- explicit naming decisions
- explicit test scenarios

over vague phase summaries or patch-order prose.

## Workflow

### 1. Ground In Existing Conventions

Before writing a detailed step plan:

1. Read the source spec.
2. Read the approved overview plan.
3. Read at least one existing implementation-plan document in the repo.
4. Inspect the code files that are likely extension points for the current step.

Match local document style where possible.

Do not invent a new doc style if the repo already has one.

### 2. Freeze The Plan Level

Confirm that this document is a detailed step plan, not an overview plan and not code implementation.

By default, a detailed step plan should contain:

- step title and purpose
- short summary
- step-level inputs and outputs
- implementation files and responsibilities
- per-file functions
- per-function responsibility
- per-function input/output contract
- schema/model descriptions and fields
- test scenarios

By default, it should not contain:

- actual code patches
- line-by-line edit order
- speculative abstractions without ownership
- repeated restatement of the entire overview plan

### 3. Lock Step Boundaries First

Before detailing functions, define:

- what this step owns
- what this step does not own
- which upstream outputs it consumes
- which downstream outputs it must produce

If ownership is fuzzy between adjacent steps, fix that before writing file/function details.

### 4. Lock File Ownership

Each file listed in the detailed step plan must have one main responsibility.

When proposing files:

- keep one main responsibility per file
- split files when one file would mix orchestration and calculation
- split files when one file would mix schema and behavior
- split files when one file would mix data extraction and rendering
- avoid catch-all `helpers` or `utils` ownership
- avoid introducing intermediate models just to group a few paths or values unless they have real semantic meaning

If a proposed file feels like over-engineering, collapse it.

### 5. Lock Function Contracts

Every function named in the detailed step plan must define:

- whether it is public or private
- its responsibility
- its inputs
- its outputs
- a short output example

Prefer the smallest stable function surface that still removes ambiguity.

When naming functions, actively avoid names that can be confused with:

- config field names
- user-facing concepts
- similar inputs from a different layer
- broader overview-plan terminology

If a function input is a derived subset or filtered variant, name it distinctly rather than reusing the broader source name.

### 6. Lock Schema Contracts

If the step introduces models, schemas, dataclasses, or pydantic models, define all of them explicitly.

For each model, include:

- why it exists
- what fields it has
- what each field means when non-obvious

Do not write “schema model” without enumerating its fields.

If a model is only a convenience grouping and has no stable meaning, prefer not introducing it.

### 7. Add Concrete I/O Examples

For every important function and model, include a compact example of the produced value.

Good examples:

- resolved path string
- validated config shape
- snapshot object shape
- aggregation output row
- manifest payload

This prevents hidden interpretation gaps without needing code.

### 8. Design Tests As Scenarios

Detailed plans must include scenario-based tests, not only module-level test categories.

Good:

- “when `output_dir` is omitted, default path resolves from checkpoint”
- “when no sold samples exist, `estimated_inventory_days` is null”
- “when two boost causes overlap, larger fraction is selected”

Weak:

- “test config”
- “test simulation”
- “test report”

## Required Sections

Unless the repo has a stronger convention, include these sections in each detailed step plan.

### Step Summary

State:

- what this step is responsible for
- what this step intentionally leaves to other steps

### Inputs / Outputs

Must define:

- step-level inputs
- step-level outputs
- short examples of major outputs

### Implementation Files And Responsibilities

Must define:

- file paths
- one-line responsibility per file

### Functions

For each file, must define:

- public functions
- private functions
- responsibility
- inputs
- outputs
- output example

### Schemas / Models

If this step introduces typed contracts, must define:

- model name
- why it exists
- fields
- non-obvious field meanings

### Test Scenarios

Must define scenario-based tests that validate the contracts and semantics of the step.

## Anti-Ping-Pong Checklist

Before finalizing, check whether the detailed step plan clearly answers all of these:

- What exactly does this step own?
- What files are added or changed in this step?
- Does each file have only one main responsibility?
- Are any proposed files clearly over-engineered?
- Are all public functions named?
- Are all important private functions named?
- Does every function have responsibility, inputs, outputs, and an output example?
- If schemas/models are introduced, are all fields listed explicitly?
- Are any names likely to be confused with config fields or broader concepts?
- Are filtered or derived inputs named distinctly from their broader sources?
- Do the tests describe concrete scenarios rather than generic categories?
- Could another engineer implement this step without reopening naming or ownership questions?

If any answer is missing, the detailed step plan is not ready.

## Decision Heuristics

When several reasonable options exist, prefer:

- fewer files with clear ownership over extra intermediate files without semantic value
- explicit schema fields over “see implementation”
- filtered-input names over overloaded source names
- step-local contracts over repo-wide abstraction guesses
- concrete scenario tests over broad test headings
- examples that show shape and semantics over prose-only descriptions

## Output Pattern

When generating a detailed step plan, prefer this sequence:

1. Read the source spec and approved overview plan.
2. Inspect relevant existing code for the current step.
3. Lock step ownership and boundaries.
4. Define step-level inputs and outputs.
5. Define implementation files and responsibilities.
6. Define functions per file.
7. Define schemas/models and their fields.
8. Define scenario-based tests.

If the user asks for review rather than creation, critique the detailed step plan using the anti-ping-pong checklist above.

## Final Check

Before considering the document complete, ask:

`Can a different engineer implement this step directly from this document without inventing function contracts, schema fields, or test scenarios?`

If the answer is no, the detailed step plan is not ready.
