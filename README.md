# sdlc-skills

A collection of AI agent skills for orchestrating SDLC workflows: project planning, implementation, bug tracking, testing, and git commits.

## Overview

`sdlc-skills` provides a set of structured, reusable skills for AI coding agents. Each skill defines a multi-phase workflow that guides your agent through a specific stage of the software development lifecycle — from initial planning all the way through to committing and shipping code.

These skills are written for [OpenCode V2](https://opencode.ai/v2/docs/) but follow a general pattern compatible with other agentic coding tools that support custom skill/instruction files.

## Skills

| Skill                  | Description                                                                             |
| ---------------------- | --------------------------------------------------------------------------------------- |
| `sdlc-plan`         | Reconciles `Requirements.md` with the codebase and generates or updates `Plan.md`       |
| `sdlc-update`       | Performs a gap assessment between `Plan.md` and source code, generating `Update.md`     |
| `sdlc-implement`    | Orchestrates sequential source code updates based on the gaps identified in `Update.md` |
| `sdlc-update-tests` | Reviews and updates the test suite to reflect recent code changes                       |
| `sdlc-bug-create`   | Triages, analyzes, and documents new issues in `Bugs.md`                                |
| `sdlc-bug-fix`      | Selects, implements, verifies, and documents fixes for items in `Bugs.md`               |
| `sdlc-commit`       | Verifies test health, manages git branching conventions, and commits changes safely     |

## Requirements (OpenCode V2)

- **Subagents.** Every skill is an orchestrator: it never edits files or runs commands itself. Work is delegated with the `subagent` tool to the built-in `general` subagent (or `explore` for read-only analysis). Leave `experimental.subagent_depth` at its default of `1`; the skills never rely on nested delegation.
- **Todo tracking.** Skills record progress with `todowrite`, which is disabled unless you opt in:

  ```jsonc
  {
    "$schema": "https://opencode.ai/config.json",
    "experimental": { "todo": true },
  }
  ```

  Without the flag the skills still work; they simply track phases in the conversation instead of a todo list.
- **Skill IDs.** In V2 the skill ID is the directory name, not the frontmatter `name` (that field is a display label). Keep the layout `skills/<skill-id>/SKILL.md`.

## Usage

### With OpenCode

1. Copy the desired skill directories into your skills directory, either global (`~/.config/opencode/skills/`) or project-local (`.opencode/skills/`), preserving the `<skill-id>/SKILL.md` layout.

2. Load a skill in one of these ways:

   - Mention it directly, e.g. `@sdlc-plan`.
   - Ask the agent to load it by ID ("load the `sdlc-plan` skill"), which uses the `skill` tool.

   Slash commands such as `/sdlc-plan` only exist if you also define a matching file under `.opencode/commands/`.

3. The agent will execute the skill's phases sequentially, delegating analysis, file changes, and shell commands to subagents as needed.

### With Other Platforms

Each skill is a self-contained markdown file with clearly defined phases and agent delegation instructions. To adapt a skill for another platform:

- Point your agent at the `SKILL.md` file as a system prompt or instruction file.
- Ensure your platform supports multi-step, sequential task execution and sub-agent delegation.
- Replace any platform-specific tool references with the equivalent tools available in your environment.

The OpenCode V2 tool names used throughout these skills are:

| Concept            | OpenCode V2 tool                                  |
| ------------------ | ------------------------------------------------- |
| Delegate work      | `subagent`                                        |
| Ask the user       | `question`                                        |
| Track progress     | `todowrite` (requires `experimental.todo: true`)   |
| Load another skill | `skill`                                           |
| Run a command      | `shell`                                           |

### Recommended Workflow Order

For a greenfield project or major feature cycle, run the skills in this order:

```
sdlc-plan → sdlc-update → sdlc-implement → sdlc-update-tests → sdlc-commit
```

Use `sdlc-bug-create` and `sdlc-bug-fix` at any point in the cycle to track and resolve issues.

## Contributing

Contributions are welcome! To add a new skill or improve an existing one:

1. Fork the repository and create a new branch.
2. Add or modify the relevant `SKILL.md` file.
3. Include a clear description in the skill's frontmatter (`name` and `description` fields).
4. Open a pull request with a brief explanation of what the skill does and why it's useful.

Please keep skills focused on a single, well-defined workflow stage.

## License

MIT
