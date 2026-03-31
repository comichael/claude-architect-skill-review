---
name: agent-review
description: Review agent code for common architectural mistakes and anti-patterns. Use this skill when the user asks to review, audit, or check agent code, agentic systems, multi-agent pipelines, or Claude API integrations. Also use when the user mentions agent review, agent audit, agent checklist, or asks "is my agent code correct". Trigger even for partial implementations or planning discussions about agent architecture.
---

# Agent Architecture Review

Scan the current project for agent-related code (API calls, tool definitions, agent loops, subagent invocations, prompt templates, error handling). Then check against the following list.

For each item, report: PASS, FAIL (with file and line), or N/A (not applicable to this project).

## Agentic Loop
- [ ] Loop controlled by `stop_reason`, not text content or content type
- [ ] `max_loops` used only as safety net, not as control mechanism
- [ ] No natural language parsing to detect completion

See references/1-agentic-architecture.md for details.

## Subagent Context
- [ ] All context passed explicitly in subagent prompts
- [ ] Context passed as structured data, not unstructured prose
- [ ] Each subagent receives only relevant context, not everything

See references/1-agentic-architecture.md for details.

## Coordinator Decomposition
- [ ] Coordinator instructed to consider breadth of decomposition
- [ ] Validation step between planning and execution

See references/1-agentic-architecture.md for details.

## Business Rule Enforcement
- [ ] Financial, security and compliance rules enforced in code, not only in prompts
- [ ] PreToolUse hooks for operations with monetary consequences
- [ ] PostToolUse hooks trimming irrelevant fields from tool results

See references/1-agentic-architecture.md for details.

## Task Decomposition
- [ ] Large inputs split into per-item passes, not processed in one call
- [ ] Separate integration pass after per-item analysis

See references/1-agentic-architecture.md for details.

## Tool Descriptions
- [ ] Every tool description is at least two sentences
- [ ] Similar tools have explicit boundary statements
- [ ] Descriptions include expected inputs and when NOT to use the tool

See references/2-tool-design.md for details.

## Tool Distribution
- [ ] No agent has more than 7 tools
- [ ] Simple lookups not routed through coordinator unnecessarily

See references/2-tool-design.md for details.

## Error Handling
- [ ] Empty results and connection failures handled differently
- [ ] Error responses include errorCategory and isRetryable
- [ ] Business errors not marked as retryable

See references/2-tool-design.md for details.

## Configuration
- [ ] Team standards in project-level config, not user-level
- [ ] Credentials use environment variable expansion, not hardcoded
- [ ] Path-specific rules used instead of duplicated directory-level config

See references/3-claude-code-config.md for details.

## CI/CD
- [ ] Claude Code invoked with -p flag in CI scripts
- [ ] Code review uses independent instance, not the generating session

See references/3-claude-code-config.md for details.

## Prompts and Schemas
- [ ] No vague instructions ("be conservative", "only if confident")
- [ ] Extraction schemas use nullable fields where source may lack data
- [ ] Few-shot examples include reasoning, not just answers

See references/4-prompt-engineering.md for details.

## Persistent Facts
- [ ] Transactional data stored in separate object, not only in conversation history
- [ ] Facts object passed in every API call

See references/4-prompt-engineering.md for details.

## Context Management
- [ ] Key findings placed at the beginning of long inputs
- [ ] Independent review instance for high-stakes output
- [ ] Long exploration sessions use scratchpad or summary checkpoints

See references/5-context-management.md for details.

## Escalation
- [ ] No sentiment-based escalation logic
- [ ] Explicit human request honored immediately without further resolution attempts

See references/5-context-management.md for details.

## Error Propagation
- [ ] Subagent failures include failure type, what was attempted, and partial results
- [ ] Pipeline does not terminate entirely on single subagent failure

See references/5-context-management.md for details.