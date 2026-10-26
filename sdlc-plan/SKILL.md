---
name: sdlc-plan
description: Use this skill to perform a gap analysis between Requirements.md, the current codebase, and existing Plan.md tasks, then generate a clean Plan.md.
---

# Skill Instructions

**Constraint:** Do not modify any source code files, configuration files, or files inside designated test directories defined in `./Testing Strategy.md`. Only modify `./Plan.md`. Use the `todowrite` tool (requires `experimental.todo: true`) to track the planning lifecycle.

⚠️ **STRICT AGENT BOUNDARY & ORCHESTRATION RULES:**
- You are acting SOLELY as an **orchestrator**. You must NOT perform direct work (file edits, code analysis, or terminal execution) in the main interface.
- All technical inspection, file updates, and shell executions MUST be delegated via the `subagent` tool.
- Call `subagent` with `agent` (`general`, or `explore` for read-only analysis), a short `description`, and a complete, self-contained `prompt`. Subagents start with fresh context, so restate every path, selection, and constraint.
- Make one `subagent` call at a time and wait for its returned summary before continuing.
- V2 subagents cannot launch their own subagents (`experimental.subagent_depth` defaults to 1). All delegation happens from this orchestrator; a subagent that hits a blocker returns and you decide the next step.
- `todowrite` is progress tracking only (requires `experimental.todo: true`). Use it to record phases and completion state; it does not dispatch work.

**EXECUTION POLICY:** Perform steps strictly in sequence. Do not spawn concurrent agents or proceed to final file generation until the analysis phase is verified.

---

## Phase 1 — Revision & Gap Analysis

Record the analysis step with `todowrite`, then call the `subagent` tool with `agent: explore` (read-only analysis) and the following prompt.

> Read `./Requirements.md` as the authoritative functional baseline.
> 
> **REVISION HISTORY ANALYSIS:**
> Parse the **Revision History** table/section at the end of `./Requirements.md`. 
> 1. Identify the current target version from the latest entry in the Revision History.
> 2. Read the current version heading and active tasks in `./Plan.md` (if it exists). 
> 3. Identify any intermediate or skipped revisions between the baseline version in `Plan.md` and the target version in `Requirements.md` (e.g., jumping from `1.1.0` directly to `1.1.2`, skipping `1.1.1`).
> 4. Compile a cumulative list of all functional, structural, and architectural changes across all intermediate revisions leading up to the target version.
> 
> **CODEBASE & EXISTING PLAN RECONCILIATION:**
> Review all source code files (excluding designated test directories defined in `./Testing Strategy.md`) to compare current codebase state against both `./Requirements.md` AND existing items in `./Plan.md`.
> - Check every action item, checklist task, and bug noted in the old `./Plan.md` directly against current source files.
> - **Flag for Exclusion:** Flag any task, bug, or requirement that has already been implemented/resolved in the code, or removed from `./Requirements.md`, so it is excluded during file generation.
> - **Flag for Retention:** Retain ONLY genuinely outstanding gaps, unverified code, structural drift, or new requirements from `Requirements.md`.
> 
> Return a clear summary of findings:
> 1. List of items from old `Plan.md` confirmed RESOLVED or OBSOLETE (to be pruned).
> 2. List of remaining/new work strictly required for target version.
> 3. Explicit notes on any multi-revision jump context.

**WAIT:** Do not proceed until the analysis step is marked complete and verified.

---

## Phase 2 — Generate/Replace Plan Artifact

Based on the verified findings from Phase 1, record the Plan generation step with `todowrite`, then call the `subagent` tool with `agent: general` and the following prompt.

> Create `./Plan.md` if it does not exist. If it exists, completely overwrite and replace its contents.
> 
> ⚠️ **PRUNING & REPLACEMENT RULE:** The file generation step prunes completed and obsolete items by completely replacing `Plan.md` with a fresh build containing ONLY active, outstanding tasks and current state assessments. Do NOT carry forward resolved tasks or historical drift notes.
> 
> Structure `./Plan.md` as follows:
> - **Header Tag:** `**Version [Target Version]**`
> - **Revision Jump Context (If applicable):** If intermediate revisions were jumped (e.g., `1.1.0` → `1.1.2`), include a sub-section explicitly listing all intermediate versions processed during this planning cycle and their primary changes.
> - **Section 1: Current State Assessment** (Verification table and remaining structural/functional drift breakdown).
> - **Section 2: Core Engineering Decisions or Filename Consistency** (Architectural alignment for all target changes).
> - **Section 3: Testing & Verification Status** (Unit, Integration, and Manual Checklists for remaining/new work using `- [ ]` checkboxes).
> - **Section 4: Exit Codes**
> - **Section 5: Security**
> - **Section 6: Dependencies**
> - **Section 7: Non-Functional Requirements**
> 
> **WAIT:** Do not consider this skill execution complete until `./Plan.md` has been successfully written and verified.