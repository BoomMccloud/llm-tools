---
description: Run multi-stage feature development pipeline on a spec file. Orchestrates first-principle, KISS, verification, TDD, implementation guide, implementation loop, and review.
---

# Feature Development Pipeline

You are running a comprehensive feature development pipeline. Process the spec file at `$ARGUMENTS` through each stage sequentially.

**CRITICAL**: Complete each stage fully before moving to the next. Present findings clearly after each stage.

**Pipeline Flow**: `first-principle → kiss → verify → tdd → guide → implement → review`

---

## Stage 1: First-Principle Analysis

**Goal**: Establish the architectural foundation using first-principles thinking.

**Key Question**: "What is the correct architecture for this problem?"

1. Read the spec file at `$ARGUMENTS`
2. Apply `/first-principle` skill to analyze:
   - Core requirements (what MUST this do?)
   - Separation of concerns
   - Single source of truth for state
   - Testable component boundaries
   - Data flow between components
3. Output a summary of architectural decisions
4. If fundamental architectural issues found, stop and report.

**Stop Condition**: Fundamental architectural issues that need user decision.

---

## Stage 2: KISS Evaluation

**Goal**: Simplify the architecture to avoid over-engineering.

**Key Question**: "Can I remove anything without breaking requirements?"

1. Apply `/kiss` skill to evaluate:
   - Unnecessary abstractions
   - Premature optimization
   - Over-engineering patterns
   - Complexity vs value tradeoffs
2. Suggest simplifications if needed
3. If major simplifications are needed, update the spec or stop for user input

**Stop Condition**: Major over-engineering that needs spec rewrite.

---

## Stage 3: Verify (Reality Check)

**Goal**: Validate the spec against the actual codebase.

**Key Question**: "Can we actually build this?"

1. Apply `/verify` skill to check:
   - File paths exist
   - Methods/functions exist with correct signatures
   - Dependencies are installed
   - Data models match
   - Naming conventions are consistent
2. Generate a verification report
3. Save the verification report to `docs/todo/<FEATURE_NAME>_verification_report.md`

**Stop Condition**: Blocking issues (referenced files/methods don't exist).

---

## Stage 4: TDD (Define Success Criteria)

**Goal**: Write failing tests that define what "done" looks like.

**Key Question**: "What behavior must pass for this feature to be complete?"

1. Use the **Task tool** to spawn the `tdd-test-writer` agent:
   - Pass the verified spec file path as input
   - The agent will write failing tests that capture required behaviors
   - Tests define WHAT the code should do, not HOW
2. Wait for the agent to complete
3. Verify tests exist and fail as expected (they should fail - nothing is implemented yet)
4. Record the test file path(s) created

**Important**: Tests drive the implementation. They are written BEFORE the implementation guide.

---

## Stage 5: Guide (Implementation Instructions)

**Goal**: Create step-by-step instructions to make the tests pass.

**Key Question**: "How do we make these tests green?"

1. Using the verified spec and the failing test files from Stage 4, generate a comprehensive implementation guide with:
   - Step-by-step instructions ordered by dependency
   - Each step tied to which test(s) it helps pass
   - Exact file locations and line numbers
   - Complete code snippets for each step
   - "Run tests after this step" checkpoints
   - Common mistakes to avoid
2. Save the implementation guide to `docs/todo/<FEATURE_NAME>_implementation_guide.md`
3. Include the test file path(s) as acceptance criteria in the guide

---

## Stage 6: Implement (Make Tests Pass)

**Goal**: Implement the feature until all tests pass.

1. Use the **Task tool** to spawn the `agent-ralph` agent with:
   - **Spec path**: The implementation guide from Stage 5
   - **Test command**: The test command for the specific test file(s) from Stage 4
2. The agent will loop until:
   - All integration tests pass
   - Typecheck passes
3. Report the final implementation status

**Stop Condition**: Max iterations reached without tests passing.

---

## Stage 7: Review (Final Quality Check)

**Goal**: Ensure the code will be understandable in 6 months.

**Key Question**: "Will this be clear to someone new in 6 months?"

1. Apply `/review` skill to evaluate:
   - **Fresh Eyes Test**: Can someone understand without asking the author?
   - **Why Not What Test**: Are non-obvious decisions explained?
   - **Mental Model Test**: Can I build a correct mental model in 5 minutes?
   - **Grep Test**: Can I find what I'm looking for?
   - **Change Confidence Test**: Could I safely modify this code?
   - **Onboarding Test**: Could a new team member work on this?
   - **2AM Debuggability Test**: Could I diagnose issues with only logs?
2. Report the 7-test score (e.g., 6/7 PASS)
3. Check that no unnecessary complexity crept in during implementation
4. Verify tests adequately cover key behaviors

**Note**: This is a final gate. Report issues but don't block (tests already pass).

---

## Pipeline Diagram

```
┌─────────────────────────────────────────────────────┐
│  1. FIRST-PRINCIPLE                                 │
│     Establish architectural foundation              │
│     Output: Architecture spec                       │
└─────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────┐
│  2. KISS                                            │
│     Simplify, remove unnecessary complexity         │
│     Output: Simplified spec                         │
└─────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────┐
│  3. VERIFY                                          │
│     Check spec against codebase reality             │
│     Output: Verification report                     │
└─────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────┐
│  4. TDD                                             │
│     Write failing tests (behavior-first)            │
│     Output: Failing test file(s)                    │
└─────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────┐
│  5. GUIDE                                           │
│     Step-by-step to make tests pass                 │
│     Output: Implementation guide                    │
└─────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────┐
│  6. IMPLEMENT (agent-ralph)                         │
│     Loop until tests pass                           │
│     Output: Passing tests + code                    │
└─────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────┐
│  7. REVIEW                                          │
│     Maintainability check (7 tests)                 │
│     Output: Review report                           │
└─────────────────────────────────────────────────────┘
                        │
                        ▼
                   ✅ COMPLETE
```

---

## Stop Conditions

The pipeline should STOP and wait for user input if:

1. **Stage 1**: Fundamental architectural issues that need user decision
2. **Stage 2**: Major over-engineering that needs spec rewrite
3. **Stage 3**: Blocking issues in verification (referenced files/methods don't exist)
4. **Stage 6**: Max iterations reached without tests passing

---

## Output Summary

After completing all stages (or stopping early), provide:

```markdown
## Pipeline Summary

**Spec File**: [path]
**Status**: COMPLETED | STOPPED_AT_STAGE_X

### Stage Results

| Stage | Name            | Status    | Output                      |
| ----- | --------------- | --------- | --------------------------- |
| 1     | First-Principle | PASS/FAIL | Architecture decisions      |
| 2     | KISS            | PASS/FAIL | Simplifications applied     |
| 3     | Verify          | PASS/FAIL | [verification report]       |
| 4     | TDD             | DONE      | [test file paths]           |
| 5     | Guide           | CREATED   | [implementation guide]      |
| 6     | Implement       | PASS/FAIL | [iterations, files changed] |
| 7     | Review          | X/7 PASS  | [maintainability notes]     |

### Final Artifacts

- Verification Report: [path]
- Test Files: [paths]
- Implementation Guide: [path]

### Next Steps

[What the user should do next, if anything]
```

---

## Key Pipeline Properties

| Property        | Value                       |
| --------------- | --------------------------- |
| Total Stages    | 7                           |
| TDD Position    | Before Guide (Stage 4)      |
| Test Purpose    | Drive implementation        |
| Guide Input     | Spec + verification + tests |
| Review Position | Final (Stage 7)             |

---

## Usage

```bash
/feature-pipeline docs/todo/FEAT47_specification.md
```
