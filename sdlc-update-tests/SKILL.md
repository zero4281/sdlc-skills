---
name: sdlc-update-tests
description: Use this skill to review and update the test suite after code changes.
---

# Skill Instructions

**Constraint:** Do not modify any core source files. Only modify files located within the test directories defined in `./Testing Strategy.md` (and `./Testing Strategy.md` itself in Phase 4). Use the `todowrite` tool (requires `experimental.todo: true`) to track all execution items.

⚠️ **STRICT AGENT BOUNDARY & ORCHESTRATION RULES:**
- You are acting SOLELY as an **orchestrator**. You must NOT perform direct work (file edits, code analysis, or terminal execution) in the main interface.
- All technical inspection, file updates, and shell executions MUST be delegated via the `subagent` tool.
- Call `subagent` with `agent` (`general`, or `explore` for read-only analysis), a short `description`, and a complete, self-contained `prompt`. Subagents start with fresh context, so restate every path, selection, and constraint.
- Make one `subagent` call at a time and wait for its returned summary before continuing.
- V2 subagents cannot launch their own subagents (`experimental.subagent_depth` defaults to 1). All delegation happens from this orchestrator; a subagent that hits a blocker returns and you decide the next step.
- `todowrite` is progress tracking only (requires `experimental.todo: true`). Use it to record phases and completion state; it does not dispatch work.

**EXECUTION POLICY:** Perform workflow steps strictly in sequence. Do not spawn concurrent agents. Wait for each returned summary before proceeding.

---

## Phase 1 — Analyze

**Constraint:** You must execute these items sequentially. Do not batch them. Wait for the summary of each step before creating the next. You must NOT read the files to perform this analysis yourself—delegate everything to a `subagent` call.

Record the following three analysis steps with `todowrite` and dispatch each through the `subagent` tool (`agent: general`, sequential calls only). 

Include the following context in every delegation prompt:

- Read `./Testing Strategy.md` to locate the designated testing folders, testing principles, consolidation/workflow rules, and execution commands.
- Extract the **primary test suite execution command**, the **full/comprehensive test suite execution command** (if defined), and any **targeted single-file runner commands** defined near the top of `./Testing Strategy.md` (or in its validation commands section). ALWAYS run tests using these designated commands. **DO NOT run tests or execute code using ad-hoc inline snippets (such as `python -c '[code]'`).**
- Only modify files inside the identified test directories.
- Read `./Requirements.md` and **specifically Section 3 (Testing & Verification Status) of `./Plan.md`** before starting. Do not review core implementation tasks outside of this section.
- Return a written summary of findings, including a list of specific fixes and consolidations needed.

### Item 1 — Coverage Gaps & Workflow Mapping

> Read `./Testing Strategy.md`. Locate the designated testing folders and the canonical workflow descriptions. Compare behaviors specified in `./Requirements.md`, Section 3 of `./Plan.md`, and the behavioral/coverage sections of `./Testing Strategy.md` against the existing test files.
> 
> For each specified behavior with missing coverage, classify how it should be addressed while maintaining the anti-redundancy and workflow-consolidation guidelines in `./Testing Strategy.md`:
> 1. **Extend Existing Workflow Test (Preferred Default):** Identify the specific existing test in the matching test file whose workflow already exercises the relevant screen, lifecycle, or request flow, and specify the exact assertions to fold into that test.
> 2. **Create New Workflow Test (Only When Distinct):** Propose a new test method/block *only* when the behavior represents a genuinely distinct workflow or isolated edge case that cannot be combined into an existing test without breaking determinism. Note how the new test should be recorded in the canonical test list in `./Testing Strategy.md`.
> 
> **Configuration Testing Rule:** Configurable settings must never be tested by checking default values on the homepage/frontend alone; they must be tested in a workflow that sets custom values on the settings page, saves, and verifies those values on the target page (per `./Testing Strategy.md`).
> 
> New or extended tests must stay within the existing file that matches their coverage area — do not create new test files unless explicitly required by `./Testing Strategy.md`. Return a written summary listing each gap, the target file and existing test to extend (or new workflow test to add), and any updates needed for the test list in `./Testing Strategy.md`.

**WAIT:** Do not proceed until Item 1 is complete and the summary is returned.

### Item 2 — Stale & Redundant Tests

> Scan the existing tests within the designated test directories for both **staleness** and **redundancy** against `./Requirements.md`, current source code, and the consolidation guidelines in `./Testing Strategy.md`:
> 1. **Stale Tests:** Identify tests that no longer match the current source code or requirements (e.g., outdated selectors, return values, removed methods, changed geometry, or superseded behavior).
> 2. **Redundant & Fragmented Tests:** Identify tests that violate the consolidation guidelines in `./Testing Strategy.md`, including:
>    - Separate single-property or single-field tests that repeat the same setup, login, form save, or HTTP request and should be combined into a unified workflow test.
>    - Tests that check hard-coded default values on the homepage/frontend when they should be setting custom values in the settings page and testing for those values.
>    - Tests that misuse client-side config overrides as a substitute for real settings persistence, or duplicate coverage already provided by another test in the suite.
> 
> Return a written summary listing each stale or redundant test group and the specific update or consolidation needed.

**WAIT:** Do not proceed until Item 2 is complete and the summary is returned.

### Item 3 — Mocking Compliance

> Read `./Testing Strategy.md` to check if a mock verification script path is defined in the configuration parameters. If a verification script is specified and exists on disk, run it using the environment's appropriate interpreter. Otherwise, manually scan the test directory files to ensure mocking, route-interception, and config-override patterns match the compliance and intercept rules defined in `./Testing Strategy.md`. Note any violations. Return a written summary listing any violations found.

---

## Phase 2 — Fix, Consolidate & Verify

Once all three analysis steps are complete and their summaries are returned, group the findings **by target test file / workflow** so related additions, updates, and consolidations in the same file are executed together cleanly. Record one `todowrite` item per target file/workflow group and dispatch each through the `subagent` tool (`agent: general`) one at a time.

**Constraint:** You must create and execute these items one at a time. Wait for the agent to confirm the fix and verification before moving to the next item.

Each delegation prompt must include:

- The specific test file and the exact workflow test(s) to extend, consolidate, modify, or remove (from the Phase 1 analysis summaries).
- The constraint: only modify files inside the project's designated test directories, and adhere strictly to the workflow-consolidation and settings-to-frontend rules in `./Testing Strategy.md`.
- **Test Command Rule:** Read the targeted single-file test runner command matching the modified test file's suite/directory (or the primary test runner command if no single-file runner is specified) in `./Testing Strategy.md` and run it against the modified test file to verify the change.
- **Iterative Verification:** After applying the changes, run the relevant test file command to verify that the consolidated/updated workflow passes cleanly. Continue adjusting the test until it passes.
- **Unresolvable Blockers & Bug Filing:** If a test fails due to a bug or limitation in core source code (which cannot be modified per constraints):
  1. Stop attempting to modify the test file.
  2. Gather the exact core code failure details, including steps to reproduce, expected vs. actual behavior, and affected source files.
  3. Formulate a structured bug summary and return it to the orchestrator, which loads the `sdlc-bug-create` skill to log the issue into `./Bugs.md`. Do not attempt to load the skill or launch agents yourself (`subagent_depth` defaults to 1).
  4. Mark the item as blocked due to an open core bug report.

**WAIT:** Do not begin Phase 3 until all Phase 2 items are marked complete or explicitly reported as blocked with a filed bug report.

---

## Phase 3 — Verify & Retest

**Constraint:** The orchestrator must not run test execution commands directly. Spawn a `general` subagent via the `subagent` tool to execute all test runs and analyze failures.

1. **Initial Verification Run:**
   Record the following item with `todowrite`, then dispatch it through the `subagent` tool (`agent: general`):
   
   > Read `./Testing Strategy.md` to extract the primary test suite execution command (e.g., from §1.1 or §10.0). Execute that exact command in the terminal. **Do not use ad-hoc inline snippets (such as `python -c`).** Analyze the full test runner output. If all tests pass, return a summary confirming 100% pass rate. If any tests fail, list every failing test name, the specific assertion failure/error output, and the file location.
   
2. **Iterative Repair Loop:**
   If there are failing tests:
   
   - For each failing test file, record a separate `todowrite` item and dispatch it through the `subagent` tool (`agent: general`). 
   - Instruct the `general` subagent to analyze the specific test failure, inspect the test file, apply the required fix within the test directory, and re-run the specific test file using the designated runner command to confirm the fix works. **Do not use `python -c` snippets.**
   - If a test cannot pass because of a defect in core source implementation code, instruct the `general` subagent to gather the details and return a structured blocker report; the orchestrator then loads the `sdlc-bug-create` skill to file the bug report in `./Bugs.md` and documents the blocker.
   - Execute these fix items one at a time.
   
   **WAIT:** Do not proceed until all testing layers pass cleanly.
   
3. **Final Full-Suite Validation:**
   Once the primary test suite passes cleanly, record a final suite validation item with `todowrite` and dispatch a `subagent` call (`agent: general`) to extract and execute the **full/comprehensive test suite execution command** defined in `./Testing Strategy.md` (if `./Testing Strategy.md` does not define a separate full/comprehensive suite command, re-run the **primary test suite execution command**). If any tests fail during this run, repeat Step 2 (Iterative Repair Loop) for each failing test file using its targeted single-file runner command, then re-run the full/comprehensive test suite execution command as the final command. 
   
**WAIT:** Repeat this verification loop until the `general` subagent explicitly confirms that both the **primary test suite** and the **full/comprehensive test suite** run and pass cleanly with zero failures (or provides a summary of remaining failures that have been successfully logged to `./Bugs.md` via `sdlc-bug-create`).

---

## Phase 4 — Update Testing Strategy

Record the following update item with `todowrite`, then call the `subagent` tool with `agent: general`:

> Update `./Testing Strategy.md` only:
> 
> 1. **Re-enumerate test counts:** Run the test enumeration commands specified in `./Testing Strategy.md` (or the test runner's `--list` / `--list-tests` flags) to obtain the exact current test counts per file and suite. Never guess or hand-estimate counts.
> 2. **Sync the canonical list of workflow tests & inventory tables:**
>    - If any new workflow test was added, or if existing tests were merged/renamed during Phase 2, update the corresponding workflow test descriptions and inventory tables in `./Testing Strategy.md` so the documented test list matches the test files on disk while preserving the guidelines that prevent redundancy.
>    - Update the per-file and total test count metrics in `./Testing Strategy.md` from the enumerated output.
> 3. **Clean up resolved coverage gaps:** Remove any entries from the coverage gaps section that are now addressed.
> 4. Do not change unrelated sections unless source code or requirement changes have made them factually incorrect.
