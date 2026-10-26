---
name: sdlc-commit
description: Use this skill to verify test health, manage git branching conventions, and commit code changes safely.
---

# Skill Instructions

**Constraint:** Do not bypass failing tests. Use the `todowrite` tool (requires `experimental.todo: true`) to manage commit lifecycle tracking.

⚠️ **STRICT AGENT BOUNDARY & ORCHESTRATION RULES:**
- You are acting SOLELY as an **orchestrator**. You must NOT perform direct work (file edits, code analysis, or terminal execution) in the main interface.
- All technical inspection, file updates, and shell executions MUST be delegated via the `subagent` tool.
- Call `subagent` with `agent` (`general`, or `explore` for read-only analysis), a short `description`, and a complete, self-contained `prompt`. Subagents start with fresh context, so restate every path, selection, and constraint.
- Make one `subagent` call at a time and wait for its returned summary before continuing.
- V2 subagents cannot launch their own subagents (`experimental.subagent_depth` defaults to 1). All delegation happens from this orchestrator; a subagent that hits a blocker returns and you decide the next step.
- `todowrite` is progress tracking only (requires `experimental.todo: true`). Use it to record phases and completion state; it does not dispatch work.

**EXECUTION POLICY:** Perform lifecycle steps strictly in sequence. If a preceding phase fails (e.g., test regressions or dirty validation issues), halt the pipeline immediately. Do not execute any subsequent git mutations or pushes.

---

## Phase 1 — Pre-Commit Testing Verification

Before attempting any version control actions, record a test verification step with `todowrite`, then call the `subagent` tool with `agent: general`.

> Read `./Testing Strategy.md` to extract the primary test suite execution command.
> Execute the full regression test suite using that exact command.
> Return the final test status to the orchestrator.
> 
> **CRITICAL:** If any tests fail, report the failures immediately and abort the workflow. Do not proceed to branch evaluation or staging under any circumstances.

**WAIT:** Do not proceed to Phase 2 until the test verification step is marked complete and runs completely green.

---

## Phase 2 — Branch Management Analysis

Once tests are verified to pass, record a branch assessment step with `todowrite`, then call the `subagent` tool with `agent: general`.

> Read the current version number from `./Plan.md`.
> Inspect the current local repository git state.
> 1. Identify the current active branch name.
> 2. If the current branch is `main`:
>    - Execute `git diff HEAD -- Bugs.md` (or check uncommitted/staged diffs for `Bugs.md`).
>    - Determine the branch type prefix: select `bugfix/` if `Bugs.md` has been modified and any bug entry was marked as completed (e.g., status updated to `RESOLVED`). Otherwise, select `feature/`.
>    - Construct a new branch using the exact format: `[feature/bugfix]/[Extracted-Version]-[short-description]` (e.g., `bugfix/v1.0.7-fix-install-titles` or `feature/v1.0.7-user-confirmation-flows`).
>    - Create and switch to this new branch.
> 3. If the repository is already checked out to a feature or bugfix branch (any branch other than `main`), continue using that active branch without creating a new one.
> 
> Return the final confirmed branch name back to the orchestrator.

**WAIT:** Do not proceed until the branch status has been confirmed and returned.

---

## Phase 3 — Stage, Document, and Commit

Record the staging and commit generation step with `todowrite`, then call the `subagent` tool with `agent: general`.

> 1. Check `git status` and scan for untracked artifacts using `git ls-files --others --exclude-standard` to ensure all relevant files are captured.
> 2. Ingest the modifications made to the codebase and auto-generate a git commit message.
> 
> **COMMIT MESSAGE CONSTRAINT:** The commit subject (first line) must be concise (maximum 72 characters) suitable for use as a Pull Request title. Optionally, include a brief body description below a blank line for necessary context. Avoid long lists or multi-paragraph blocks.
> 
> 3. Stage the files, execute the git commit, and push the active branch to the remote repository.
>    **Note:** V2 may return an `ask` permission prompt for `git push`. Surface it to the user and wait rather than retrying or working around it.
> 
> Return a final success confirmation including the commit message and targeted push URL.