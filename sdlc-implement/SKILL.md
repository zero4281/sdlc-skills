---
name: sdlc-implement
description: Use this skill to orchestrate and execute source code updates based on the gaps identified in Update.md, verifying work item by item and conducting full suite validation.
---

# Skill Instructions

**Constraint:** Do not modify any markdown documentation files (`.md`), **with the sole exception of updating `./Plan.md` to mark completed items (`- [ ]` to `- [x]`)**. Do not modify, add, or delete any files inside designated test directories as defined in `./Testing Strategy.md`. Use the `todowrite` tool (requires `experimental.todo: true`) to manage high-level implementation tracking.

⚠️ **STRICT AGENT BOUNDARY & ORCHESTRATION RULES:**
- You are acting SOLELY as an **orchestrator**. You must NOT perform direct work (file reading/analysis, code edits, or terminal execution) in the main interface.
- All technical inspection, work breakdown analysis, file updates, and shell executions MUST be delegated via the `subagent` tool.
- Call `subagent` with `agent` (`general`, or `explore` for read-only analysis), a short `description`, and a complete, self-contained `prompt`. Subagents start with fresh context, so restate every path, selection, and constraint.
- Make one `subagent` call at a time and wait for its returned summary before continuing.
- V2 subagents cannot launch their own subagents (`experimental.subagent_depth` defaults to 1). All delegation happens from this orchestrator; a subagent that hits a blocker returns and you decide the next step.
- `todowrite` is progress tracking only (requires `experimental.todo: true`). Use it to record phases and completion state; it does not dispatch work.

**EXECUTION POLICY:** 
1. **AUTONOMOUS INITIATION:** Upon invocation of this skill, immediately begin Phase 1. Do NOT ask the user what to orchestrate or wait for additional input. The presence of `./Update.md` and `./Plan.md` in the workspace is your instruction to begin.
2. **Delegation Mandatory:** Every phase—including Phase 1 analysis—must be recorded with `todowrite` and dispatched through the `subagent` tool (`agent: general`). Do not read or inspect project files directly as the orchestrator.
3. **Sequential Work Loop:** Code updates in Phase 2 must be performed strictly one work item at a time. Do not batch modifications or initiate a new sub-item until the previous item has been fully implemented, manually verified, and marked complete in `./Plan.md` by the `general` subagent.

---

## Phase 1 — Implementation Breakdown Analysis

Upon skill invocation, immediately record the implementation breakdown step with `todowrite`, then call the `subagent` tool with `agent: general`.

Instruct the `general` subagent to:
1. Read `./Requirements.md`, `./Plan.md`, and `./Testing Strategy.md` to establish complete technical context.
2. Read `./Update.md` to extract all required code updates, additions, modifications, and features targeted for removal.
3. Compare `Update.md` against `./Plan.md` to map out discrete, ordered logical units of work matching the tasks in `./Plan.md`.
4. Return a clear, numbered sequence of Work Items to the orchestrator.

**WAIT:** Do not proceed to Phase 2 until the `general` subagent completes the breakdown analysis and returns the ordered list of Work Items. Maintain this list within the orchestrator's execution loop state; do not write it to an external file.

---

## Phase 2 — Sequential Implementation & Per-Item Verification Loop

For each Work Item identified in Phase 1, record a `todowrite` item, then dispatch it with the `subagent` tool (`agent: general`).

**Constraint:** You must execute these Work Items strictly one at a time. Wait for confirmation, test output, and a written summary of changes from the `general` subagent before creating the task for the next item.

Each `subagent` prompt for a Work Item must instruct the `general` subagent to perform the following sequence:

1. **Execute Code Update:**
   - Apply the specific target feature addition, modification, or removal instructions across source files (referencing line ranges if available from `Update.md`).
   - Enforce the explicit constraint: Do not modify markdown files (except marking `./Plan.md`) or any testing environment paths.
2. **Per-Item Manual Dynamic Verification:**
   - Immediately perform manual dynamic verification specifically targeting the item's changes.
   - Read the **Manual Dynamic Testing** section of `./Testing Strategy.md` to extract mandatory testing constraints, execution environments, sandbox/isolation protocols, and interaction procedures.
3. **Mark Complete in `./Plan.md`:**
   - Upon successful manual dynamic verification, update `./Plan.md` by changing the corresponding task checkbox from `- [ ]` to `- [x]`.

**REALLOCATION LOOP:** If the `general` subagent fails to update or verify a component cleanly, re-verify the code diff or error output, supply the specific compilation or logical failure, and re-dispatch the fix in a new `subagent` call before moving to the next item in `./Plan.md`.

---

## Phase 3 — Integrated Retesting & Unit Suite Validation

Once all individual work items from Phase 2 are complete and marked off in `./Plan.md`, record the final validation step with `todowrite`, then call the `subagent` tool with `agent: general` to perform integrated dynamic testing and run the automated unit test suite.

The orchestrator must instruct the `general` subagent to read `./Testing Strategy.md` to extract mandatory testing constraints, execution environments, session logging/capture patterns, special sandbox/isolation protocols, interactive verification lifecycles, and automated unit test commands.

Instruct the `general` subagent to execute the complete verification lifecycle according to the procedures outlined in `./Testing Strategy.md`, enforcing the following rules as clear, explicit constraints in the prompt:

- **CRITICAL — Environment Isolation & Safety Gates:** Parse `./Testing Strategy.md` for any "Special Case" testing requirements or destructive workflows. Set up the specified isolation environment (such as a temporary sandbox directory) and execute mandatory validation gates before launching the execution session if required.
- **STATE SYNCHRONIZATION POLICY:**
  1. **Pre-Test Sync:** Copy fresh, edited source files from the active working repository into the sandbox environment.
  2. **Post-Failure Sync:** If verification fails, apply code fixes directly to the **active working repository**, *not* inline inside the sandbox.
- **DUAL VERIFICATION EXECUTION:**
  1. **Full Manual Dynamic Retest:** Execute end-to-end manual dynamic testing across all integrated changes using session runner procedures specified in `./Testing Strategy.md` to confirm overall system stability.
  2. **Automated Unit Test Suite Run:** Execute the project's full automated unit test suite using the exact command specified in `./Testing Strategy.md`. Ensure all pre-existing and new unit tests pass cleanly.
- **Scratch Artifacts:** State captures, execution logs, and output dumps must be written strictly to scratch/temporary locations (e.g., `/tmp/`).
- **Session Cleanup:** The application instance or interactive test session must be explicitly terminated and cleaned up once final verification states are captured.
- **INLINE ERROR REPAIR:** If manual dynamic retesting or automated unit testing reveals failures during execution, apply necessary fixes directly to the source code files and repeat the verification loop until all manual dynamic captures and unit test runs confirm stable, intended behavior.

Review the resulting artifacts returned by the `general` subagent:
- Verify that specific feature changes, success states, or outputs expected from `Update.md` requirements are visible in captured application states.
- Confirm that the application handles input correctly, processes dynamic workflows as expected, and that all automated unit tests pass.

**CRITICAL:** Fixes for any behavioral bugs, execution errors, or functional flaws discovered here must be applied strictly to the source code. Never modify automated test files, test configurations, or paper over issues by adjusting interactive execution scripts.

**WAIT:** If dynamic testing or unit tests reveal failures or incorrect behavior, the orchestrator must record a follow-up item with `todowrite` and dispatch a new `subagent` call (`agent: general`) to repair the source code. Repeat Phase 3 until all verification artifacts confirm stable, intended behavior.