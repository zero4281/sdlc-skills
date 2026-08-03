# Coding Rules & Execution Guardrails (OpenCode)

## CRITICAL EXECUTION POLICY: Terminal & Tool Loops

1. **Strict Command Deduplication:**
   - NEVER execute the exact same shell command twice in a row.
   - If a bash command returns output (stdout or stderr), you MUST analyze the output text before taking your next action.
   - You may only re-run a previously failed or executed command if you have modified its flags, arguments, or underlying files based on the output.

2. **Acknowledge Output & State Transitions:**
   - After any bash command execution, your immediate next step must be to assess the state:
     - **Command succeeded?** Proceed to the next logical step in the task.
     - **Command failed / error thrown?** Diagnose the error and attempt a fix or alternative command. Do NOT blindly re-run the failed command.
     - **Command hung / timed out?** Terminate the process and attempt a non-blocking or targeted alternative.

3. **Disallowed Command Hallucinations & Tool Rules:**
   - NEVER attempt to run `task`, `todowrite`, `todoread`, or `opencode` as shell/bash commands.
   - `todowrite` is an internal OpenCode tool (`todowrite`), NOT a CLI executable.
   - If you need to manage task items or step tracking, call the native `todowrite` tool function directly.
   - Do NOT assume interactive session runners or terminal tools (such as `tmux`) are globally required unless specified in the project's `./Testing Strategy.md`. Parse interactive runner protocols directly from `./Testing Strategy.md`.

---

## FILE EDIT POLICY: Match Anchors & Edits

1. **Keep `oldString` Ultra-Short (1-2 Lines Max):**
   - NEVER pass full code blocks or multi-line paragraphs inside `oldString`.
   - Your `oldString` must contain ONLY 1 to 2 lines of unique code to pinpoint the edit location.
   - Example:
     ```json
     // BAD (Do NOT do this):
     "oldString": "def calculate_total(items):\n    total = 0\n    for item in items:\n        total += item.price\n    return total"

     // GOOD (Do this):
     "oldString": "def calculate_total(items):"
     ```

2. **Always Re-Read First:**
   - Execute `read_file` on the target area immediately before calling an edit. Never edit from distant memory.
   - Copy the exact characters directly from the most recent `read_file` output block.

3. **Fallback Procedure on Match Failure:**
   - If you get "Could not find oldString in the file", STOP immediately.
   - Re-read the file to check line numbers and whitespace.
   - Retry using a single unique line as the `oldString` anchor.

4. **Disambiguate Duplicate Anchors:**
   - If the target `oldString` line appears multiple times in the file (e.g., `return True` or `pass`), expand `oldString` to 2–3 lines *only* until it includes a completely unique structural element (like a specific variable name or function header) to ensure the edit lands in the correct location.

5. **Strict Indentation Mirroring:**
   - When writing the `newString` replacement block, perfectly mirror the exact spaces or tabs observed in the preceding `read_file` output. Do not assume or alter the indentation level unless explicitly instructed to refactor it.