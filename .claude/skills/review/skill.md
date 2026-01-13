---
name: review
description: Final quality check for maintainability and correctness after implementation. Use after all tests pass to evaluate whether code will be understandable 6 months later. Runs at the end of the feature pipeline.
---

# Review (Final Quality Check)

This skill is the final step in the feature development pipeline. It evaluates whether the implemented code will be understandable 6 months from now.

## Position in Pipeline

```
first-principle → kiss → verify → tdd → guide → implement → [YOU]
```

By the time you run:

- Architecture has been designed and simplified
- Spec has been verified
- Tests have been written (TDD)
- Implementation guide has been created
- **Implementation is complete and all tests pass**

Your job: **Final quality gate before the feature ships.**

## Key Question

**"Will this be clear to someone new in 6 months?"**

## When to Use This Skill

**Trigger conditions:**

- After implementation is complete (all tests passing)
- PR review for significant features
- User asks "will this make sense later?" or "is this maintainable?"
- Evaluating code that feels clever or non-obvious
- Final check before merging

**Initial Assessment:**

Explain that you'll evaluate the implementation against the "6-Month Test" - imagining you've completely forgotten the context and are reading this code for the first time.

## The 6-Month Test Framework

### 1. The "Fresh Eyes" Test

**Core Question: "Can someone understand this without asking the author?"**

**What to Check:**

- **Names tell the story** - Do function/variable/file names explain intent?
  - Bad: `handleData()`, `processStuff()`, `utils.ts`
  - Good: `validateUserInput()`, `calculateTaxFromSubtotal()`, `priceFormatters.ts`

- **Flow is traceable** - Can you follow execution without jumping around?
  - Bad: 7 files to understand one user action
  - Good: Related logic lives together, clear entry points

- **Behavior is predictable** - Does code do what its name suggests?
  - Bad: `getUser()` that also updates a cache and logs analytics
  - Good: `getUser()` returns a user, that's it

**Red Flags:**

- Comments that explain WHAT (code should do that)
- Magic numbers or strings without explanation
- Acronyms or abbreviations that aren't universally known
- "Helper" or "Utils" files with unrelated functions

**Green Flags:**

- Code reads like prose
- Tests serve as documentation
- Clear separation between "what" (business logic) and "how" (infrastructure)

### 2. The "Why Not What" Test

**Core Question: "Are non-obvious decisions explained?"**

**What to Check:**

- **Decisions are documented** - Why this approach over alternatives?
  - Bad: Complex code with no explanation
  - Good: Comment or doc explaining the tradeoff

- **Constraints are visible** - Are limitations/requirements noted?
  - Bad: Workaround code that looks wrong
  - Good: `// API returns max 100 items, so we paginate`

- **Historical context preserved** - Why does this weird thing exist?
  - Bad: Legacy code that no one dares touch
  - Good: `// Workaround for Safari bug, see issue #123`

**Documentation Levels:**

1. **Code comments** - For "why" at the line level
2. **Function/class docs** - For contracts and edge cases
3. **README files** - For architectural decisions
4. **Spec documents** - For feature-level context

**Red Flags:**

- Complex logic with no explanation
- Workarounds without links to issues
- "TODO: fix later" without context
- Configuration values without explanation

**Green Flags:**

- Spec documents that survive implementation
- Comments explaining business rules, not code mechanics
- Links to external docs/issues where relevant

### 3. The "Mental Model" Test

**Core Question: "Can I build a correct mental model in 5 minutes?"**

**What to Check:**

- **Clear entry points** - Where does execution start?
  - Bad: Scattered initialization across files
  - Good: Single entry point, clear initialization

- **Obvious state ownership** - Who owns what data?
  - Bad: State scattered across components/modules
  - Good: Clear single source of truth, documented

- **Predictable patterns** - Same problems solved the same way?
  - Bad: Three different error handling approaches
  - Good: Consistent patterns throughout

**Complexity Budget:**

Every feature has a "complexity budget" - how much a reader can hold in their head:

| Complexity          | Budget  |
| ------------------- | ------- |
| Files to understand | 3-5 max |
| State locations     | 1-2 max |
| Abstraction layers  | 2-3 max |
| Concepts to learn   | 3-5 max |

**Red Flags:**

- Multiple valid interpretations of how code works
- State that can be modified from many places
- Patterns that differ from rest of codebase
- Circular dependencies

**Green Flags:**

- "Aha!" moment comes quickly
- State flows in one direction
- Consistent with codebase conventions

### 4. The "Grep Test"

**Core Question: "Can I find what I'm looking for?"**

**What to Check:**

- **Searchable names** - Unique, greppable identifiers
  - Bad: `status`, `data`, `handler`, `type`
  - Good: `InterviewStatus`, `BlockCompletionData`, `onBlockTransition`

- **Colocated related code** - Things that change together live together
  - Bad: Types in `/types`, logic in `/utils`, components in `/components`
  - Good: Feature folder with all related code

- **Discoverable structure** - File names match what's inside
  - Bad: `helpers.ts` with 20 unrelated functions
  - Good: `priceCalculation.ts` with price calculation functions

**Red Flags:**

- Generic names used everywhere
- `index.ts` files with lots of logic
- Re-exports that obscure location
- Deeply nested folder structures

**Green Flags:**

- Can find any function in < 30 seconds
- File names are descriptive
- Related code is adjacent

### 5. The "Change Confidence" Test

**Core Question: "Could I safely modify this code?"**

**What to Check:**

- **Blast radius is clear** - What else might break?
  - Bad: Changing one thing ripples everywhere
  - Good: Clear boundaries, obvious dependencies

- **Tests explain behavior** - What should this code do?
  - Bad: No tests, or tests that test implementation
  - Good: Tests describe behavior, serve as spec

- **Rollback is possible** - Can we undo this safely?
  - Bad: Migrations that can't be reversed
  - Good: Feature flags, reversible changes

**Red Flags:**

- "Don't touch this" warnings
- No test coverage
- Implicit dependencies (globals, singletons)
- Tightly coupled modules

**Green Flags:**

- High test coverage with descriptive test names
- Clear module boundaries
- Changes can be isolated

### 6. The "Onboarding" Test

**Core Question: "Could a new team member work on this?"**

**What to Check:**

- **Setup is documented** - How do I run this?
- **Architecture is explained** - How do pieces fit together?
- **Vocabulary is defined** - What do domain terms mean?
- **Examples exist** - How has similar work been done?

**Red Flags:**

- Tribal knowledge required
- "Ask X, they know how this works"
- Setup requires multiple undocumented steps
- Domain terms used without definition

**Green Flags:**

- README with quick start
- Architecture diagrams or docs
- Glossary of domain terms
- Similar features to reference

### 7. The "2AM Debuggability" Test ⭐

**Core Question: "Could I diagnose a production issue at 2am with only logs?"**

This test is inspired by FEAT54's "Principle 7" - the gold standard for production debugging.

**What to Check:**

- **Error messages include full context** - Operation, IDs, hints
  - Bad: `Error: Not found`
  - Good: `[WorkerService] Block 3 not found for interview abc-123. This usually means the block wasn't created during interview setup.`

- **Success/failure is explicit** - Not just errors, log positive outcomes too
  - Bad: Silent success, only logs on failure
  - Good: `[WorkerService] ✓ Block feedback generated for block-1`

- **Visual markers for scanning** - Symbols that stand out in log streams
  - Use ✓ for success, ✗ for failure
  - Use CRITICAL for urgent issues
  - Use WARNING for concerning but non-fatal issues

- **Recovery paths documented** - What should the operator do?
  - Bad: `Error: Feedback generation failed`
  - Good: `✗ CRITICAL: Feedback generation failed. Transcript already stored, retries will be skipped. Manual intervention needed.`

**Red Flags:**

- Generic error messages
- Silent failures or silent successes
- No context about what operation was attempted
- No hints about what might be wrong
- Logs that require reading code to understand

**Green Flags:**

- Errors include operation name, all relevant IDs, and context
- Success explicitly logged with ✓ symbol
- Failure explicitly logged with ✗ symbol and severity
- Recovery/retry strategy documented in error message
- Log prefixes for easy filtering (e.g., `[ServiceName]`)

**The 2AM Checklist:**

When evaluating code or specs, ask:

- [ ] Can I understand what failed from the error message alone?
- [ ] Do I know which entity/record was involved? (IDs logged)
- [ ] Can I quickly scan logs for success/failure? (✓/✗ symbols)
- [ ] Do I know if this requires immediate action? (CRITICAL markers)
- [ ] Is the recovery path clear? (retry safe? manual intervention needed?)
- [ ] Can I filter logs by component? (`[ServiceName]` prefix)

**Template: Production-Quality Error Handling**

```typescript
// 2AM-Debuggable error handling
console.log(`[ServiceName] Starting expensive operation for resource ${id}`);
try {
  await expensiveOperation(id);
  console.log(`[ServiceName] ✓ Operation completed for resource ${id}`);
} catch (error) {
  console.error(
    `[ServiceName] ✗ CRITICAL: Operation failed for resource ${id}. ` +
      `Previous state preserved. Safe to retry. ` +
      `Hint: Check if dependency service is available.`,
    {
      resourceId: id,
      operation: "expensiveOperation",
      error: error instanceof Error ? error.message : String(error),
      stack: error instanceof Error ? error.stack : undefined,
      // Include any relevant context
      userId: context.userId,
      timestamp: new Date().toISOString(),
    },
  );
  throw new Error(
    `[ServiceName] Failed to process resource ${id}: ${error instanceof Error ? error.message : String(error)}`,
  );
}
```

**Template: Idempotency with Safety Checks**

```typescript
// Check for existing work (idempotency)
if (resource.processedData) {
  // CRITICAL: Verify downstream work was also completed
  const dependentWorkStatus = resource.dependentWork
    ? "✓ exists"
    : "✗ MISSING (manual fix needed)";

  console.log(
    `[ServiceName] Resource ${resource.id} already processed. ` +
      `Dependent work status: ${dependentWorkStatus}. ` +
      `Skipping duplicate submission.`,
  );

  // This distinguishes:
  // - Normal skip (everything is fine)
  // - Problematic skip (dependent work failed, needs manual fix)
  return;
}
```

**Template: Transaction Boundary Comments**

```typescript
// Store core data in transaction (fast, atomic)
await db.$transaction([
  db.resource.update({
    /* ... */
  }),
  db.auditLog.create({
    /* ... */
  }),
]);

// Generate derived data (outside transaction - long-running AI call)
// CRITICAL: If this fails, resource will have core data stored
// but derived data will be missing. This is intentional - allows retry.
// Status will remain IN_PROGRESS until this completes successfully.
console.log(`[ServiceName] Generating derived data for ${resourceId}`);
try {
  await generateDerivedData(resourceId);
  console.log(`[ServiceName] ✓ Derived data generated for ${resourceId}`);
} catch (error) {
  console.error(
    `[ServiceName] ✗ CRITICAL: Derived data generation failed. ` +
      `Core data stored successfully. Safe to retry.`,
    { resourceId, error },
  );
  throw error;
}
```

## The Maintainability Evaluation Process

### Step 1: Understand the Scope

Review the spec/code and identify:

- What is the feature/change?
- What files/modules are involved?
- What are the key decisions made?

### Step 2: Apply the Seven Tests

For each test, evaluate:

- **PASS**: Maintainable, will be understood later
- **WARNING**: Some concerns, may need attention
- **FAIL**: Will cause confusion, needs improvement

### Step 3: Score and Prioritize

| Score    | Meaning                           |
| -------- | --------------------------------- |
| 7/7 PASS | Exemplary - use as template       |
| 5-6 PASS | Ship it                           |
| 3-4 PASS | Good, minor improvements optional |
| 1-2 PASS | Address failures before shipping  |
| 0 PASS   | Major rework needed               |

### Step 4: Provide Recommendations

For each WARNING or FAIL:

- What specifically is the problem?
- What's the risk in 6 months?
- What's the concrete fix?

## Spec Document Evaluation

When evaluating a spec document (like FEAT54), check:

### Document Structure

- [ ] Problem statement is clear
- [ ] Root cause is explained (not just symptoms)
- [ ] Solution approach is justified
- [ ] Alternatives were considered
- [ ] Files to modify are listed
- [ ] Acceptance criteria are testable
- [ ] TL;DR or summary section (for long specs)

### Code Changes Clarity

- [ ] Before/after code examples
- [ ] Clear diff of what changes
- [ ] No ambiguous instructions
- [ ] Edge cases addressed
- [ ] Error handling specified

### Future Reader Value

- [ ] Explains WHY this approach
- [ ] Links to related issues/specs
- [ ] Documents tradeoffs made
- [ ] Survives as useful reference

### 2AM Debuggability (Gold Standard)

Specs should include debugging requirements. Check if the spec specifies:

- [ ] **Error message requirements** - What context must errors include?
- [ ] **Logging strategy** - What gets logged at each step?
- [ ] **Visual markers** - Use of ✓/✗/CRITICAL symbols specified?
- [ ] **Recovery paths** - How should failures be handled?
- [ ] **Idempotency safety** - Checks for dependent work completion?
- [ ] **Transaction boundaries** - What happens if later steps fail?
- [ ] **Debugging checklist** - Requirements for implementation?

**Example from FEAT54 (Exemplary):**

```markdown
## Principle 7: 2AM Debuggability

**Decision**: All code must be debuggable at 2am with only logs

**Implementation**:

1. Error messages include context: All errors include operation, IDs, and hints
2. Explicit success/failure logging: Try/catch around expensive operations
3. Idempotency safety checks: Verify feedback exists when skipping
4. Transaction boundary comments: Explain what happens if later steps fail

## Debugging Checklist

- [ ] Include operation name
- [ ] Include all relevant IDs
- [ ] Include hints about what might be wrong
- [ ] Wrap expensive operations in try/catch
- [ ] Log success with ✓ symbol
- [ ] Log failure with ✗ symbol and CRITICAL marker
- [ ] Verify dependent work when skipping (idempotency)
- [ ] Comment transaction boundaries with failure scenarios
```

If a spec lacks these, it's a **WARNING** - implementation may not be production-ready.

## Example Evaluation

### Example: FEAT44 Session Architecture Simplification

**Fresh Eyes Test:** PASS

- Clear problem statement with specific bug description
- Before/after code examples show the change
- Diagrams explain the flow

**Why Not What Test:** PASS

- Root cause analysis explains the hidden state problem
- First-principle analysis documents WHY this design
- Tradeoffs explicitly discussed

**Mental Model Test:** PASS

- Single concept: "make driver stateless"
- State topology diagram shows before/after
- Flow diagrams are clear

**Grep Test:** PASS

- Command names are unique (`CONNECT_FOR_BLOCK`)
- File locations explicitly listed
- Searchable terms used

**Change Confidence Test:** PASS

- Tests listed with grep patterns
- Acceptance criteria are checkable
- Single PR approach reduces risk

**Onboarding Test:** PASS

- Appendix provides architecture comparison
- First-principle checklist serves as learning material
- Self-contained document

**Overall: 6/7 - This spec will be maintainable**

The document itself will serve as valuable context 6 months later. A new developer could read this and understand both the change AND the architectural principles behind it.

_(Note: Would be 7/7 with explicit 2AM debuggability requirements)_

---

### Example: FEAT54 Consolidate Worker Logic ⭐ GOLD STANDARD

**Fresh Eyes Test:** PASS

- Clear function names: `getWorkerContext`, `submitWorkerTranscript`
- Service layer pattern explicitly documented
- Before/after code examples show exact changes
- `[WorkerService]` log prefix for easy grep

**Why Not What Test:** PASS - EXEMPLARY

- Four critical decisions explicitly documented with rationale (lines 109-163):
  - Why tRPC pattern wins (documented, tested, working)
  - Why await feedback (reliability > speed)
  - Why simple idempotency (no schema changes needed)
  - Why protobuf stays in endpoints (transport concern)
- Transaction boundaries explicitly commented
- Failure recovery paths documented

**Mental Model Test:** PASS

- Single concept: "Extract to service layer, follow dumb driver pattern"
- Clear layering: Worker → Endpoint → Service → Database
- State ownership clear: Database is single source of truth
- Complexity budget: 4 files, 1 state location, 2 layers (within budget)

**Grep Test:** PASS

- Unique searchable names
- Verification commands included in spec (lines 1182-1185)
- Log prefix `[WorkerService]` for filtering
- All worker logic in one file

**Change Confidence Test:** PASS

- Comprehensive unit tests specified (lines 877-1037)
- Integration tests should pass unchanged
- No schema changes (safe rollback)
- Incremental deployment plan (Phase 1-4)

**Onboarding Test:** PASS

- Implementation order clear (4 phases)
- Testing strategy provided
- Links to related features (FEAT43, 47, 48, 51, 53)
- Service layer pattern documented with examples

**2AM Debuggability Test:** PASS - EXEMPLARY ⭐

This is where FEAT54 truly excels:

- **Principle 7 explicitly addresses 2AM scenario** (lines 134-151)
- **Debugging checklist mandatory for implementation** (lines 787-871)
- **Error messages include**: operation, IDs, context, hints
- **Explicit success/failure logging**: ✓/✗ symbols, CRITICAL markers
- **Idempotency safety**: Verifies dependent work exists (lines 357-367)
- **Transaction boundaries commented**: Explains failure scenarios (lines 444-447)
- **Visual scanning**: Symbols for quick log parsing
- **Recovery paths**: Every error explains what to do next

**Code Examples from FEAT54:**

```typescript
// Error with full context (line 349)
throw new Error(
  `[WorkerService] Block ${blockNumber} not found for interview ${interviewId}. ` +
    `This usually means the block wasn't created during interview setup.`,
);

// Explicit logging with visual markers (lines 381-400)
console.log(`[WorkerService] Generating feedback for block ${block.id}`);
try {
  await generateBlockFeedback(db, block.id);
  console.log(`[WorkerService] ✓ Block feedback generated for ${block.id}`);
} catch (error) {
  console.error(
    `[WorkerService] ✗ CRITICAL: Feedback generation failed for block ${block.id}. ` +
      `Transcript already stored, retries will be skipped. Manual intervention needed.`,
    { interviewId, blockId: block.id, blockNumber, error, stack },
  );
  throw new Error(
    `Feedback generation failed for block ${block.id}: ${error.message}`,
  );
}

// Idempotency with safety check (lines 357-367)
if (block.transcript) {
  const feedbackStatus = block.feedback
    ? "✓ exists"
    : "✗ MISSING (manual fix needed)";
  console.log(
    `[WorkerService] Transcript already exists for block ${block.id}. ` +
      `Feedback status: ${feedbackStatus}. Skipping duplicate submission.`,
  );
  return;
}

// Transaction boundary comment (lines 444-447)
// Generate feedback (outside transaction - long-running Gemini call)
// CRITICAL: If this fails, interview will have transcript + endedAt,
// but status remains IN_PROGRESS. This is intentional - allows retry.
```

**Overall: 7/7 - EXEMPLARY - Use as template**

This spec represents the gold standard for maintainability documentation:

1. **Explicitly addresses 2AM scenario** - Rare to see this in specs
2. **Debugging checklist ensures implementation quality** - Not just guidelines, but requirements
3. **Error messages as documentation** - Include hints, context, and recovery paths
4. **Visual markers for production** - ✓/✗/CRITICAL for quick log scanning
5. **Safety checks built in** - Idempotency verifies dependent work
6. **Root cause analysis** - Not just symptoms, explains WHY
7. **Self-contained and comprehensive** - Can be understood without additional context

**Key Takeaway:**

The "2AM Debuggability" principle (Principle 7) should be **adopted for all critical features**. It ensures that production issues can be diagnosed and resolved quickly, even when the original author is unavailable.

**Sections to copy for future specs:**

1. Principle 7: 2AM Debuggability (lines 134-151)
2. Debugging Checklist (lines 787-871)
3. Transaction boundary comments with failure scenarios
4. Error message templates with context and hints
5. Idempotency safety checks that verify dependent work

## Anti-Patterns to Flag

### 1. "Clever" Code

```typescript
// Bad: Clever, hard to understand later
const result = items.reduce(
  (a, b) => ({ ...a, [b.id]: (a[b.id] || 0) + b.val }),
  {},
);

// Good: Clear, boring, maintainable
const result: Record<string, number> = {};
for (const item of items) {
  result[item.id] = (result[item.id] || 0) + item.val;
}
```

### 2. Implicit Dependencies

```typescript
// Bad: Requires knowing global state exists
function processOrder() {
  const user = getCurrentUser(); // Where does this come from?
  const cart = getCart(); // Global state?
}

// Good: Explicit dependencies
function processOrder(user: User, cart: Cart) {
  // Clear what's needed
}
```

### 3. Naming That Requires Context

```typescript
// Bad: Need context to understand
const flag = true;
const temp = getData();
handleIt(temp);

// Good: Self-documenting
const isUserAuthenticated = true;
const userPreferences = await fetchUserPreferences();
applyUserPreferences(userPreferences);
```

### 4. Missing "Why" Comments

```typescript
// Bad: No explanation for non-obvious code
setTimeout(callback, 100);

// Good: Explains the why
// Delay needed because DOM needs to repaint before measurement
// See: https://github.com/org/repo/issues/123
setTimeout(callback, 100);
```

## Integration with Other Skills

This skill works well with:

- **KISS**: Simple code is more maintainable. Use KISS first, then verify maintainability.
- **First-Principle**: Good architecture is inherently more maintainable. First-principle ensures the design is sound, maintainability ensures it's understandable.
- **Verify**: After verifying a spec is implementable, check if it's maintainable.

## The Maintainability Mantra

**"Write code for the person who will read it in 6 months - it might be you, and you will have forgotten everything."**

Remember:

- Names matter more than comments
- Boring is better than clever
- Explicit is better than implicit
- Consistency beats perfection
- Documents decay slower than memory
