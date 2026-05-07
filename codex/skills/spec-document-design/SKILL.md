---
name: spec-document-design
description: Design, review, and finalize specification documents from requirements, existing code, and prior docs. Use when drafting a new spec, critically reviewing an existing spec for ambiguity or implementation risk, or integrating user decisions back into the document so the final spec is implementable, internally consistent, and easy to verify.
---

# Spec Document Design

Use this skill when a task involves writing, reviewing, or tightening a specification document.

This skill covers one continuous workflow:

1. Draft the spec from requirements and source-of-truth artifacts.
2. Critically review the spec for ambiguity, conflict, and implementation risk.
3. Integrate decisions back into the body so the final document is self-contained.

## When To Use

Use this skill when any of the following are true:

- The user wants a new spec, design doc, technical spec, requirements doc, evaluation spec, migration spec, or implementation-facing document.
- A spec already exists and needs critical review before implementation.
- Requirements are partially described in chat and partially encoded in code or prior docs.
- The user wants to reduce future back-and-forth by tightening unclear sections now.

Do not use this skill for lightweight prose editing where implementation semantics do not matter.

## Core Rule

Prefer code and existing enforced contracts over stale prose.

When a spec describes behavior that already exists somewhere, find the source of truth first:

- code
- schemas
- tests
- configs
- reports
- prior specs

If prose and code disagree, document the code-backed behavior unless the user explicitly wants to change it.

## Workflow

### Phase 1: Build Context

Before writing or rewriting the spec:

1. Inspect existing spec format.
   - Match the local doc style instead of inventing a new structure.
2. Find source-of-truth artifacts.
   - Read the minimum code and docs needed to understand contracts.
3. Separate stable facts from policy decisions.
   - Stable facts come from code, schemas, or existing enforced behavior.
   - Policy decisions require explicit wording in the spec.

State the source-of-truth inputs near the top of the document.

### Phase 2: Draft The Spec

Default structure:

- Summary
- Source Of Truth
- Goals
- Non-Goals
- Key Decisions
- Evaluation or Validation
- Outputs or Artifacts
- Verification

Adapt section names to the project, but keep the logic.

When drafting:

- Define units, indexing, and time axes explicitly.
- Define how internal values differ from user-facing values.
- Define null, zero, default, fallback, and masked cases explicitly.
- Define per-item and aggregate metric formulas separately.
- Keep the document implementation-facing, not aspirational.

### Phase 3: Critically Review The Spec

After drafting, read the document as an implementer.

Look for places where two reasonable engineers could implement different behavior.

Specifically search for:

- `null` vs `0` vs omitted vs default confusion
- user-facing values vs internal values vs metric values
- per-sample vs per-item vs aggregate metric confusion
- input representation vs display representation confusion
- missing stop conditions
- missing ordering rules
- missing tie-breaking rules
- unclear sampling or aggregation rules
- sections that duplicate the same rule in multiple places
- prose that conflicts with source-of-truth code

When reviewing, produce only high-signal findings:

- the issue
- why it can lead to implementation drift
- the exact section or line range
- the smallest fix that removes ambiguity

### Phase 4: Resolve Open Questions

When the spec still has unresolved choices:

1. Collect only the questions that materially affect implementation.
2. Phrase them as concrete decisions, not vague brainstorm prompts.
3. Prefer a short list of high-impact questions over exhaustive uncertainty.

Good examples:

- Should missing predictions be represented as `null` or excluded rows?
- Should the aggregate metric ignore `null` rows or treat them as a fallback value?
- Which artifact is the source of truth when prose and code disagree?

Avoid low-value questions that can be derived from context.

### Phase 5: Integrate Decisions

When the user answers open questions:

1. Update the body first.
2. Remove or rewrite stale wording that now conflicts with the decision.
3. Collapse duplicate rules into one canonical statement where possible.
4. Keep a temporary "confirmed decisions" section only as a checkpoint.
5. Once all confirmed items are clearly reflected in the body, reduce that section to a brief status note or remove it if the project does not need it.

Do not leave the true rule only in a “confirmed decisions” appendix.

## Design Rules

### 1. Distinguish Value Layers

If the spec contains computed values, separate:

- internal simulation value
- stored value
- displayed value
- metric value

If these can differ, say so explicitly.

### 2. Define Missingness Precisely

Whenever a field can be absent, define:

- when it is `null`
- when it is `0`
- when it is not produced at all
- whether missing rows are excluded from metrics

### 3. Name Decision Boundaries

A good spec freezes the fragile boundaries:

- state transitions
- ordering within a time bucket
- sampling domain
- aggregation rule
- stopping rule
- tie-break rule
- source-of-truth priority

### 4. Keep Metrics Mechanical

For every metric, define:

- item-level numerator and denominator
- aggregate-level formula
- which rows are ignored
- which fallback values are not allowed

### 5. Preserve Reader Trust

Do not claim the document is final if key rules still live only in chat.

The final spec should be implementable without needing the conversation history.

## Review Checklist

Use this checklist before considering the spec ready:

- Are source-of-truth files named explicitly?
- Are all important axes, units, and indices defined?
- Are stop conditions explicit?
- Are all nullable fields defined with exact semantics?
- Are display values separated from computation values?
- Are aggregate metrics defined mechanically?
- Are unresolved decisions isolated into a short list?
- After user answers, were those decisions integrated into the body?
- Is any rule duplicated in multiple places with slightly different wording?

## Output Pattern

When using this skill in a conversation, prefer this sequence:

1. Draft the spec.
2. List only the highest-impact open questions.
3. After answers arrive, integrate them into the body.
4. Re-read the spec critically.
5. Provide a decision-to-location mapping when useful so the user can verify the integration quickly.

## Default Style

- Write spec documents in Korean by default, unless the user explicitly requests another language or the repository has a stronger existing language convention for that document set.
- Keep identifiers, code symbols, file paths, metric names, formulas, and external product/library names in their original language when translating would reduce implementation clarity.
- Concise, implementation-facing prose
- Explicit formulas where ambiguity matters
- Flat lists over nested prose when enumerating rules
- Clear separation of goals, rules, and validation

## Anti-Patterns

Avoid these mistakes:

- Writing policy decisions without checking code-backed behavior
- Leaving key decisions only in chat, not the document
- Using “appropriate”, “reasonable”, or “as needed” where exact behavior matters
- Mixing sample-level and aggregate-level metric definitions
- Repeating the same rule in summary, body, and appendix with slight wording drift

## Final Check

Before handing the spec off for planning or implementation, ask:

`Can someone implement this correctly without reading the chat history?`

If the answer is no, the spec is not done.
