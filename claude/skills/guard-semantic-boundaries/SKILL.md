---
name: guard-semantic-boundaries
description: Default review skill for modifying existing implementations where a change may alter the meaning, representation, ownership, or consumers of data. Use whenever Claude patches or reviews existing logic unless the change is purely local and non-behavioral, and especially when code touches data flow, transforms, encoders, parsers, serializers, caches, shared return values, adapters, or any producer-consumer boundary where representation mismatches can cause subtle bugs.
---

# Guard Semantic Boundaries

## Overview

Use this skill to prevent bugs caused by passing the wrong representation of data across module boundaries. Focus on representation clarity, minimal inputs, existing converters, consumer impact, and regression guards before changing code.

## Workflow

1. Name the semantic stages first.
   - Write down the relevant representations for the change, such as source, validated, canonical, parsed, indexed, serialized, cached, transport, UI-facing, or runtime-ready.
   - For each candidate input/output, state which stage it represents now and which stage the consumer expects.

2. Define the minimal required inputs.
   - Do not start with the largest shared object.
   - List the exact data the new logic needs: for example identifiers, counts, masks, schema fragments, timestamps, flags, handles, or one derived field.
   - Prefer passing those narrow values over reusing a broad aggregate that carries unrelated semantics.

3. Reuse the stage owner.
   - Find the existing transformer, parser, encoder, mapper, adapter, serializer, or validator that already owns the conversion between representations.
   - Prefer calling that API directly over extracting converted data indirectly from another object.
   - If the conversion has no clear owner, identify that as a design gap before patching around it.

4. Audit shared consumers before changing meaning.
   - If a variable, return value, or cached artifact is shared, search for every consumer before changing its meaning.
   - Do not silently repurpose a shared object from one stage to another.
   - If one consumer needs a different stage, create a new value with a new name instead of changing the contract of the old one.

5. Check whether the failure is really one level up.
   - When an exception appears, do not stop at the immediate guard or range check.
   - Ask why this representation reached this function at all.
   - Fix the semantic mismatch at the boundary when possible, not only the symptom.

6. Add guard tests around the boundary.
   - Add at least one test that proves the intended consumer receives the right stage.
   - Add at least one regression test that proves a non-target consumer or disabled path remains unchanged.
   - Prefer tests that would fail if a broad shared object were repurposed by mistake.

## Review Questions

Use these questions before finalizing a patch:

- What semantic stage does each value represent right now?
- Does the consumer expect that exact stage?
- Am I passing a whole object where a smaller value would do?
- Is there already a transformer that should own this conversion?
- Will this change alter the meaning of a shared variable or return value?
- Which other consumers need to be checked before I change that contract?
- What regression test proves unrelated paths still behave the same?

## Output Shape

When using this skill during implementation or review, produce a short note in this order:

1. Semantic stages involved
2. Minimal inputs required
3. Existing transformer or owner to reuse
4. Shared consumers at risk
5. Guard tests to add or verify

## Mnemonic

Use this checkpoint when the change feels slippery:

`Semantic Stage -> Minimal Inputs -> Existing Transformer -> Consumer Audit -> Guard Tests`
