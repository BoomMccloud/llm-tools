---
name: guide
description: Produce step-by-step implementation instructions to make failing tests pass. Use AFTER TDD tests are written. Takes verified spec + failing test files as input and generates foolproof instructions with exact file locations, code snippets, and common mistakes to avoid. A junior developer should be able to follow this guide and make all tests green.
---

# Guide

Transform verified specs and failing tests into step-by-step implementation guides that make tests pass.

## Position in Pipeline

```
first-principle → kiss → verify → tdd → [YOU] → implement → review
```

By the time you run:

- Architecture has been designed (first-principle)
- Design has been simplified (kiss)
- Spec has been verified against codebase (verify)
- **Failing tests exist that define success criteria (tdd)**

Your job: **Create instructions to make those tests pass.**

## When to Use This Skill

Use this skill when:

- You have a verified feature spec
- You have **failing tests** from the TDD step
- You need to hand off implementation to a developer or agent
- You want step-by-step instructions with exact code snippets
- You need verification gates at each step

## Required Inputs

1. **Verified Feature Spec**: The spec document after verification
2. **Verification Report**: Output from the verify skill
3. **Failing Test File(s)**: Tests from the TDD step that define success criteria
4. **Codebase Access**: Ability to read the actual repository files

## Key Question

**"How do we make these tests green?"**

## Reference Example

**IMPORTANT**: Before generating an implementation guide, read `.claude/skills/guide/example.md` for a complete reference implementation.

## Guide Workflow

### Phase 1: Analyze Tests and Reconcile

Before writing instructions, understand what the tests require:

#### 1.1 Read Each Failing Test

For each test in the test file(s):

- What behavior does it expect?
- What function/method is being called?
- What inputs are provided?
- What outputs/side effects are expected?

#### 1.2 Map Tests to Implementation Steps

```
Test: "should create interview with blocks"
├── Calls: interviewRouter.create()
├── Expects: Interview record created
├── Expects: 3 Block records created
└── Implementation needed:
    ├── Update create procedure
    ├── Add block creation logic
    └── Return interview with blocks
```

#### 1.3 Fix Spec Errors from Verification

For each blocking issue in the verification report:

- Determine the correct file path, method name, or signature
- Note the correction for the implementation guide
- Read the actual file to understand context

#### 1.4 Check for Duplication

**CRITICAL**: Before any "create new" instruction, search for existing implementations.

```bash
# Search for similar functionality
grep -rn "createInterview\|interviewCreate" src/ --include="*.ts"

# Search for existing patterns to follow
find src -name "*Interview*" -o -name "*Block*"
```

If similar functionality exists:

- DO NOT instruct to create duplicate code
- Instruct to use or extend existing functionality
- Note in the guide: "We're using existing X instead of creating new"

#### 1.5 Map Exact Locations

For each change, read the actual file and identify:

```
File: src/server/api/routers/interview.ts
├── Line 1-10: Imports
├── Line 12: Router declaration
├── Line 15-45: create procedure ← MODIFY THIS
├── Line 47-62: get procedure
└── Line 64: Export
```

Record:

- Exact line number for insertion
- 5 lines of context before
- 5 lines of context after
- Any imports needed (and where to add them)

### Phase 2: Generate Implementation Guide

Produce a markdown file with this structure:

---

## Implementation Guide Format

````markdown
# Implementation Guide: [Feature Name]

**Based on Spec**: [spec filename]
**Verification Report**: [report filename]
**Test File(s)**: [test file path(s)]
**Generated**: [date]

---

## Overview

### What You're Building

[2-3 sentences explaining the feature in plain English]

### Success Criteria

**The implementation is complete when these tests pass:**

```bash
pnpm test [test-file-path]
```

| Test Name                                | What It Verifies                               |
| ---------------------------------------- | ---------------------------------------------- |
| `should create interview with blocks`    | Create procedure returns interview with blocks |
| `should reject unauthenticated requests` | Auth middleware working                        |

### Files You Will Modify

| File                                          | Action | Summary                 |
| --------------------------------------------- | ------ | ----------------------- |
| `src/server/api/routers/interview.ts`         | Modify | Update create procedure |
| `src/server/api/services/interviewService.ts` | Create | Add business logic      |

### Out of Scope - DO NOT MODIFY

These files are **not part of this task**:

- Files not listed above
- Database migrations (separate ticket)
- Unrelated modules

---

## Prerequisites

Before starting:

```bash
# Verify tests exist and fail
pnpm test [test-file-path]
# Should see failing tests - this is expected!

# Verify build works
pnpm build
```

---

## Step 1: [First Implementation Step]

### Goal

[What this step accomplishes - tied to which test(s) it helps pass]

**This step helps pass**: `test name here`

### File

`src/server/api/routers/interview.ts`

### Find This Location

Open the file and navigate to **line 45**. You should see:

```typescript
// Line 43
export const interviewRouter = createTRPCRouter({
// Line 44
  create: protectedProcedure
// Line 45
    .input(createInterviewSchema)
```

### Action: [Add/Modify Code]

**Add after line 50:**

```typescript
// Create blocks for the interview
const blocks = await ctx.db.block.createMany({
  data: template.blocks.map((b, index) => ({
    interviewId: interview.id,
    blockNumber: index + 1,
    topic: b.topic,
  })),
});
```

### Required Import

At the top of the file, add:

```typescript
import { type Block } from "@prisma/client";
```

### Common Mistakes

#### Mistake 1: Forgetting to await

```typescript
// WRONG - not awaited
const blocks = ctx.db.block.createMany({...});

// CORRECT - awaited
const blocks = await ctx.db.block.createMany({...});
```

### Verify This Step

```bash
pnpm test [test-file-path] --testNamePattern="should create interview"
```

If the test still fails, check:

1. Did you add the code at the correct location?
2. Did you add the required import?
3. Are there TypeScript errors? Run `pnpm typecheck`

---

## Step 2: [Next Step]

[Continue pattern for each step...]

---

## Final Verification

### Run All Tests

```bash
pnpm test [test-file-path]
```

**All tests must pass.**

### Run Type Check

```bash
pnpm typecheck
```

**No errors should appear.**

### Run Linter

```bash
pnpm lint
```

**Fix any lint errors.**

---

## Troubleshooting

### Error: "Test times out"

**Cause**: Async operation not completing

**Fix**: Ensure all database operations are awaited

### Error: "Expected X but received Y"

**Cause**: Return value doesn't match test expectation

**Fix**: Check the test to see exact expected shape, match it

---

## Pre-Submission Checklist

- [ ] All tests pass: `pnpm test [test-file-path]`
- [ ] No TypeScript errors: `pnpm typecheck`
- [ ] No lint errors: `pnpm lint`
- [ ] Only modified files listed in this guide
````

---

### Phase 3: Quality Checks

Before delivering the implementation guide, verify:

#### 3.1 Test Alignment

For every step in the guide:

- [ ] Identify which test(s) this step helps pass
- [ ] Verify the step is necessary (tied to a failing test)
- [ ] Remove steps not needed to pass tests (avoid over-engineering)

#### 3.2 Code Snippet Accuracy

For every code snippet:

- [ ] Actually read the target file
- [ ] Verify imports match existing patterns
- [ ] Verify types exist and are spelled correctly
- [ ] Verify method signatures match interfaces

#### 3.3 Line Number Accuracy

For every line reference:

- [ ] Open the actual file
- [ ] Go to that line number
- [ ] Verify the context matches what you wrote
- [ ] Note if earlier steps will shift these numbers

#### 3.4 Common Mistakes

Include mistakes based on:

- Verification report findings (wrong method names, signatures)
- Codebase patterns (import paths, naming)
- What could cause the specific tests to fail

---

## Writing Guidelines

### Be Test-Driven

Every instruction should tie back to making a test pass:

| Bad               | Good                                                         |
| ----------------- | ------------------------------------------------------------ |
| "Add the method"  | "Add this method to pass the `should create interview` test" |
| "Update the code" | "This change makes `should return blocks` pass"              |

### Be Explicit

| Vague              | Explicit                                                           |
| ------------------ | ------------------------------------------------------------------ |
| "Add the method"   | "Add the following method at line 48, after the `findById` method" |
| "Import the error" | "Add `import { ValidationError } from '../errors';` at line 3"     |

### Visual Markers

Use consistently:

- **File**: File paths
- **Find This Location**: Where to navigate
- **Action**: Code to add/modify
- **Required Import**: Imports needed
- **Common Mistakes**: Pitfalls
- **Verify This Step**: Test command to run

### Line Number Strategy

Since line numbers shift:

1. **Order edits bottom-to-top** when possible
2. **Warn about shifts**: "Note: After this step, line numbers below will increase by ~15"
3. **Use anchors**: "After the `findById` method" in addition to line numbers
4. **Show context**: 5 lines before/after

### Code Snippet Rules

1. **Complete implementations only** - never partial code with "add implementation here"
2. **Show context** - what comes before and after
3. **Mark new vs existing** - use comments to clarify
4. **Match codebase style** - formatting, naming, patterns

## Exit Criteria

**A junior developer could follow this guide and make all tests pass without asking questions.**

The guide is complete when:

- [ ] Every failing test has steps to make it pass
- [ ] No extra steps beyond what tests require
- [ ] Code snippets are complete and accurate
- [ ] Line numbers and file paths verified
- [ ] Common mistakes documented for each step
