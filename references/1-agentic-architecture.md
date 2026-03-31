## Agentic Loop Control

The agent loop must always be controlled by `stop_reason` from the API response.

- `stop_reason == "tool_use"` → execute the tool, return the result, continue the loop
- `stop_reason == "end_turn"` → stop

Do not use `max_loops` as a control mechanism. A hardcoded number does not know whether the task needs 3 or 15 steps.

`max_loops` is only acceptable as a safety net against infinite loops. Set it high enough that it never triggers during normal operation. Example: `max_loops = 50`.

Flag as violation:
- Loop that terminates based on text content (e.g. "if response contains 'done'")
- Loop that terminates because the response contains text (Claude can return text and tool_use in the same response)
- Loop that checks `response.content[0].type == "text"` as a completion signal (a response can contain both a text block and a tool_use block — checking for text presence misses the tool call)
- `max_loops` set below 20 without clear justification in the code


## Subagent Context Isolation

Subagents do not share memory. They do not inherit the coordinator's conversation history. They do not know what other subagents have done.

Every piece of information a subagent needs must be explicitly included in its prompt. If subagent B needs results from subagent A, the coordinator must pass those results in subagent B's prompt.

Use structured data formats for context passing. Separate content from metadata (source URLs, document names, timestamps) to preserve attribution across agents.

Flag as violation:
- Subagent prompt that references prior findings without those findings being included in the prompt
- Subagent that is expected to "know" what another subagent produced without explicit handoff
- Context passed as unstructured prose instead of structured data (e.g. raw summary instead of claim + source pairs)
- Coordinator that passes all available context to every subagent instead of filtering for relevance


## Coordinator Decomposition

When a coordinator breaks a broad task into subtasks, it determines the scope of the entire pipeline. If the decomposition is too narrow, every downstream agent executes correctly on an incomplete plan. The output looks complete but misses entire areas.

Always trace incomplete results upward to the coordinator's decomposition before investigating subagents.

Mitigations:
- Instruct the coordinator to list all relevant subtopics before assigning work
- Require the coordinator to flag uncertainty about completeness
- Use an independent validation agent to review the plan before execution starts

Flag as violation:
- Coordinator with no explicit instruction to consider breadth of decomposition
- Missing validation step between planning and execution
- Debugging subagents when the root cause is the coordinator's scope


## Hooks vs Prompts

Prompt-based guidance (instructions in the system prompt) works most of the time. Programmatic enforcement (hooks that block tool execution) works every time.

Decision rule: if a single failure costs money, creates a security incident, or violates compliance — use programmatic enforcement. For formatting preferences and style guidelines, prompts are fine.

### PreToolUse hooks
Intercept outgoing tool calls before execution. Use to block or redirect operations that violate business rules.

Example: block refunds above a threshold, require identity verification before account changes, enforce manager approval for sensitive operations.

### PostToolUse hooks
Intercept tool results after execution but before the model processes them. Use to normalize data formats, strip unnecessary fields, or log changes.

Example: convert timestamps to a consistent format, remove internal fields the model does not need, log file modifications.

Flag as violation:
- Financial, security or compliance rules enforced only via system prompt
- Missing PreToolUse hook for operations with monetary consequences
- Tool results passed directly to model without trimming irrelevant fields
- Business rules described as "the model should never..." instead of implemented as code


## Task Decomposition

Two patterns for breaking work into subtasks:

**Fixed sequential pipeline:** Steps are known upfront. Example: analyze each file individually, then run a cross-file integration pass. Best for predictable, structured tasks.

**Dynamic adaptive decomposition:** Subtasks are generated based on what is discovered at each step. Best for open-ended investigation where the shape of the problem is unknown.

### Attention dilution

Processing too many items in a single pass produces inconsistent results. The model gives detailed feedback on some items and misses obvious issues in others. It may flag a pattern as problematic in one file while approving identical code in another.

Fix: multi-pass architecture. Run per-item analysis first for consistent depth, then a separate integration pass to catch cross-item issues.

Flag as violation:
- Agent processing 10+ files or documents in a single API call without splitting
- Code review or analysis with no separate cross-file integration pass
- Large batch processing with no per-item analysis step before synthesis


## Session Management

Three options for continuing work across sessions:

- **--resume:** Prior context is still valid and files have not changed significantly.
- **fork_session:** Need to explore divergent approaches from a shared baseline.
- **Fresh start with summary injection:** Tool results are stale, files have changed, or context has degraded.

When resuming after code changes, either inform the agent about specific file changes or start fresh with a summary. Reasoning from stale tool results produces contradictory advice.