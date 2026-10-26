---
name: sdlc-update
description: Use this skill to perform a gap assessment between Requirements.md, Plan.md, and the source code, generating or replacing Update.md.
---

# Skill Instructions

**Constraint:** Do not modify any source code files, configuration files, or files inside `./Tests/`. Only modify `./Update.md`. Do not include test coverage gaps or test file changes in this artifact. Use the `todowrite` tool (requires `experimental.todo: true`) to track the gap analysis lifecycle.

⚠️ **STRICT AGENT BOUNDARY & ORCHESTRATION RULES:**
- You are acting SOLELY as an **orchestrator**. You must NOT perform direct work (file edits, code analysis, or terminal execution) in the main interface.
- All technical inspection, file updates, and shell executions MUST be delegated via the `subagent` tool.
- Call `subagent` with `agent` (`general`, or `explore` for read-only analysis), a short `description`, and a complete, self-contained `prompt`. Subagents start with fresh context, so restate every path, selection, and constraint.
- Make one `subagent` call at a time and wait for its returned summary before continuing.
- V2 subagents cannot launch their own subagents (`experimental.subagent_depth` defaults to 1). All delegation happens from this orchestrator; a subagent that hits a blocker returns and you decide the next step.
- `todowrite` is progress tracking only (requires `experimental.todo: true`). Use it to record phases and completion state; it does not dispatch work.

**EXECUTION POLICY:** Perform steps strictly in sequence. Do not spawn concurrent agents or proceed to final file generation until the complete gap assessment analysis has been returned and verified.

---

## Phase 1 — Technical Gap Assessment

Record the assessment step with `todowrite`, then call the `subagent` tool with `agent: explore` (read-only analysis) and the following prompt.

> Read `./Requirements.md` to establish the absolute, authoritative functional baseline and ultimate source of truth for the project.
> **IMPORTANT:** Parse the **Revision History** table at the end of `./Requirements.md`. Identify all revisions spanning from the current implementation baseline up to the target version (including any skipped or intermediate versions like `1.1.1` $\rightarrow$ `1.1.2`). Build a cumulative list of all required changes across these revisions.
> 
> Read `./Plan.md` to establish the active development baseline, explicitly ignoring Section 3 (Testing & Verification Status). Treat Requirements.md as a strict override if any contradictions exist between the Plan and the Requirements.
> Check if `./Update.md` currently contains either stale placeholder text: "Update.md is stale — re-run /sdlc-update." (V1 form) or "Update.md is stale — re-run the sdlc-update skill." (V2 form). If either is present, ignore the current content of Update.md completely.
> Review all local source code files, explicitly ignoring any files inside designated test directories defined in `./Testing Strategy.md`.
> 
> Perform a comprehensive multi-way gap assessment across the cumulative Requirements delta, the Plan, and the active codebase to identify:
> 1. Required features from all target/intermediate revision specifications or plan that are missing, incomplete, or partially implemented in the source code.
> 2. Implemented features currently in the source code that violate constraints or are explicitly omitted by Requirements.md (features to prune).
> 3. Any functional discrepancies or "drift" where Plan.md claims a requirement is complete, but the actual code implementation falls short of the Requirements.md specification.
> 
> Return a clear structural breakdown of required updates and code to prune.

**WAIT:** Do not proceed until the gap assessment step is marked complete and the technical summary is returned.

---

## Phase 2 — Generate/Replace Update Artifact

Based on the verified findings from Phase 1, record the update generation step with `todowrite`, then call the `subagent` tool with `agent: general` and the following prompt.

> Create `./Update.md` if it does not exist. If it does exist, completely overwrite and replace its entire contents.
> Write the gap assessment results into the file using the following layout:
> - A high-level Summary of the required alignment, explicitly calling out any multi-revision jumps processed (e.g., Target `1.1.2` including skipped `1.1.1` deltas).
> - An explicit section detailing "Implemented but Non-Required: Features to Remove" (specifying file targets, line ranges, and rationale based on Requirements.md).
> - A structured compliance table mapping out the codebase status against core functional specifications in Requirements.md and non-testing sections of Plan.md.
> - A clear set of immediate Next Steps for implementation.
> 
> CRITICAL: Ensure no test-related work items or test suite gaps are written to this file.

**WAIT:** Do not consider this skill execution complete until `./Update.md` has been successfully written and verified.