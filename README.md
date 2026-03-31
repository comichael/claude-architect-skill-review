# claude-architect-skill-review

A Claude Code skill that reviews agent code for common architectural mistakes and anti-patterns. Based on the five domains of the [Claude Certified Architect - Foundations](https://medium.com/data-science-collective/the-complete-claude-architect-study-guide-with-code-and-tutor-prompts-01f524e95c92) study guide.

## What it does

Scans your project for agent-related code and checks it against a checklist of 30+ items covering:

- Agentic loop control (stop_reason, max_loops)
- Subagent context isolation and structured handoff
- Coordinator decomposition breadth
- Business rule enforcement (hooks vs prompts)
- Tool descriptions and distribution
- Error handling (error categories, retryable flags, empty vs failed)
- Configuration (project-level vs user-level)
- Prompt quality (explicit criteria, nullable schemas, few-shot examples)
- Context management (frontloading, independent review, checkpoints)
- Error propagation and escalation patterns

## Output

For each item, the skill reports:

- **PASS** - code follows the pattern correctly
- **FAIL** - issue found, with file path and line reference
- **N/A** - not applicable to this project

Ends with a prioritized summary of what to fix first.

## Installation

Copy the `agent-review` folder into your project's `.claude/skills/` directory:

```
your-project/
└── .claude/
    └── skills/
        └── agent-review/
            ├── SKILL.md
            └── references/
                ├── 1-agentic-architecture.md
                ├── 2-tool-design.md
                ├── 3-claude-code-config.md
                ├── 4-prompt-engineering.md
                └── 5-context-management.md
```

## Usage

In Claude Code, say:

- "review my agent code"
- "check my agent architecture"
- "run agent review"

Or invoke directly with `/agent-review` if configured as a slash command.

## Language support

The skill checks architectural patterns, not syntax. It works with any language (Python, TypeScript, C#, etc.).

## Credits

Inspired by the Claude Certified Architect study guide by [Paolo Perrone](https://medium.com/data-science-collective/the-complete-claude-architect-study-guide-with-code-and-tutor-prompts-01f524e95c92) and the official [CCA-F exam guide](https://www.anthropic.com).
