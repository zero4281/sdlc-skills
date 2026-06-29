# sdlc-skills

A collection of AI agent skills for orchestrating SDLC workflows: project planning, implementation, bug tracking, testing, and git commits.

## Overview

`sdlc-skills` provides a set of structured, reusable skills for AI coding agents. Each skill defines a multi-phase workflow that guides your agent through a specific stage of the software development lifecycle — from initial planning all the way through to committing and shipping code.

These skills are designed to work with [OpenCode](https://opencode.ai/) but follow a general pattern compatible with other agentic coding tools that support custom skill/instruction files.

## Skills

| Skill                  | Description                                                                             |
| ---------------------- | --------------------------------------------------------------------------------------- |
| `project-plan`         | Reconciles `Requirements.md` with the codebase and generates or updates `Plan.md`       |
| `project-update`       | Performs a gap assessment between `Plan.md` and source code, generating `Update.md`     |
| `project-implement`    | Orchestrates sequential source code updates based on the gaps identified in `Update.md` |
| `project-update-tests` | Reviews and updates the test suite to reflect recent code changes                       |
| `project-bug-create`   | Triages, analyzes, and documents new issues in `Bugs.md`                                |
| `project-bug-fix`      | Selects, implements, verifies, and documents fixes for items in `Bugs.md`               |
| `project-commit`       | Verifies test health, manages git branching conventions, and commits changes safely     |

## Usage

### With OpenCode

1. Copy the desired `SKILL.md` files into your project's skills directory (e.g., `.opencode/skills/`).

2. Invoke a skill by name from within your OpenCode session:
   
   ```
   /project-plan
   ```

3. The agent will execute the skill's phases sequentially, delegating file analysis and code changes to sub-agents as needed.

### With Other Platforms

Each skill is a self-contained markdown file with clearly defined phases and agent delegation instructions. To adapt a skill for another platform:

- Point your agent at the `SKILL.md` file as a system prompt or instruction file.
- Ensure your platform supports multi-step, sequential task execution and sub-agent delegation.
- Replace any platform-specific tool references (e.g., `todowrite`, `AskUserQuestion`) with the equivalent tools available in your environment.

### Recommended Workflow Order

For a greenfield project or major feature cycle, run the skills in this order:

```
project-plan → project-update → project-implement → project-update-tests → project-commit
```

Use `project-bug-create` and `project-bug-fix` at any point in the cycle to track and resolve issues.

## Contributing

Contributions are welcome! To add a new skill or improve an existing one:

1. Fork the repository and create a new branch.
2. Add or modify the relevant `SKILL.md` file.
3. Include a clear description in the skill's frontmatter (`name` and `description` fields).
4. Open a pull request with a brief explanation of what the skill does and why it's useful.

Please keep skills focused on a single, well-defined workflow stage.

## License

MIT
