## CLAUDE.md Hierarchy

Three levels, from personal to shared:

- **User-level (~/.claude/CLAUDE.md):** Applies only to you. Not version-controlled. Not shared via git.
- **Project-level (.claude/CLAUDE.md):** Applies to everyone. Version-controlled. Shared.
- **Directory-level (subdirectory CLAUDE.md):** Applies when working in that specific directory.

Team-wide standards must live in project-level config. If a new team member gets inconsistent output, the first thing to check is whether the rules are in user-level instead of project-level.

Use @import to reference external files from CLAUDE.md. Use .claude/rules/ for topic-specific rule files.

Flag as violation:
- Team coding standards in user-level config
- No project-level CLAUDE.md in a team repository
- Single monolithic CLAUDE.md instead of modular organization when file exceeds 100 lines


## Path-Specific Rules

Rules files in .claude/rules/ with YAML frontmatter glob patterns:
```yaml
---
paths: ["**/*.test.tsx"]
---
```

These match files across the entire codebase. Directory-level CLAUDE.md only covers one directory. For conventions that apply to files spread across many directories (e.g. test files), path-specific rules are the only clean solution.

Path-scoped rules load only when editing matching files, reducing irrelevant context and token usage.

Flag as violation:
- Identical CLAUDE.md files duplicated across multiple directories
- Test conventions in root CLAUDE.md that only apply to test files
- Path-specific rules with overly broad globs that match unrelated files


## Plan Mode vs Direct Execution

- **Plan mode:** Complex tasks, multiple valid approaches, architectural decisions, multi-file modifications. Explore before committing.
- **Direct execution:** Clear scope, known solution. Single-file bug fix, adding a validation conditional.

The Explore subagent isolates verbose discovery output from the main conversation. Without it, multi-phase tasks exhaust the context window before the actual work begins.

Common hybrid: plan mode for investigation, direct execution for implementing the plan.

Flag as violation:
- Direct execution on multi-file refactoring or migration tasks
- Plan mode on a single-line bug fix with a clear stack trace


## Skills and Commands

- **.claude/commands/**: Project-scoped, shared via version control.
- **~/.claude/commands/**: Personal, not shared.
- **.claude/skills/ with SKILL.md**: On-demand invocation with configuration.

Key skill frontmatter options:
- **context: fork** — runs in isolated sub-agent context. Verbose output stays contained.
- **allowed-tools** — restricts which tools the skill can use.
- **argument-hint** — prompts for required parameters when invoked without arguments.

Skills are on-demand, task-specific workflows. CLAUDE.md is always-loaded, universal standards. Do not mix them.

Flag as violation:
- Task-specific procedures in CLAUDE.md instead of a skill
- Universal coding standards in a skill instead of CLAUDE.md
- Skill that produces verbose output without context: fork
- Skill with no allowed-tools restriction when it performs destructive operations


## CI/CD Integration

The `-p` flag runs Claude Code in non-interactive mode. Without it, the CI job hangs waiting for input.

Use `--output-format json` with `--json-schema` for machine-parseable findings.

The same Claude session that generated code is less effective at reviewing it. It retains reasoning context that makes it less likely to question its own decisions. Use an independent review instance.

When re-running reviews after new commits, include prior findings and instruct Claude to report only new or unresolved issues. Duplicate comments erode developer trust.

Flag as violation:
- CI script invoking Claude Code without -p flag
- CI code review using the same session that generated the code
- Review output in plain text instead of structured JSON when consumed by automation
- Re-review that duplicates previously reported findings