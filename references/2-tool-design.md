## Tool Descriptions

Tool descriptions are the primary mechanism Claude uses to select which tool to call. Not the function name. Not routing logic. The description text.

A good description includes: what the tool does, expected inputs, example queries it handles, and explicit boundaries against similar tools.

When two tools are being confused, fix the descriptions first. Not few-shot examples (wrong root cause), not a routing classifier (over-engineered), not tool consolidation (different problem).

Flag as violation:
- Tool description shorter than two sentences
- Two or more tools with overlapping descriptions and no explicit boundary statements
- Routing classifier or few-shot examples used to fix tool selection before descriptions have been improved
- Description that omits input format, expected parameters, or when NOT to use the tool


## Tool Distribution

Each agent should have 4-5 tools scoped to its role. More than that degrades selection reliability.

For high-frequency simple operations, give the agent a constrained tool directly instead of routing through the coordinator. Route only complex cases through the coordinator.

Flag as violation:
- Agent with more than 7 tools
- Simple lookups routed through coordinator when a scoped tool on the agent would eliminate round trips
- All tools given to all agents regardless of role


## tool_choice

Three modes:

- **auto:** Model decides whether to call a tool or return text. Default for general operation.
- **any:** Model must call a tool but chooses which one. Use for guaranteed structured output.
- **forced (specific tool):** Model must call this exact tool. Use to enforce mandatory first steps.

Flag as violation:
- Mandatory first step (e.g. identity verification) without forced tool_choice
- Use of "any" where a specific tool should be forced
- No explicit tool_choice setting when structured output is required


## Structured Error Responses

Four error categories with different handling:

- **Transient:** Timeout, service unavailable. Retryable.
- **Validation:** Bad input, wrong format. Fix input, then retry.
- **Business:** Policy violation (e.g. refund exceeds limit). Not retryable. Needs alternative workflow.
- **Permission:** Access denied. Needs escalation or different credentials.

Critical distinction: an empty result from a successful query means "data does not exist" — do not retry. A failed connection means "could not check" — retry is appropriate.

Error metadata should include: errorCategory, isRetryable boolean, and a human-readable description.

Flag as violation:
- Error handling that treats empty results and connection failures identically
- Missing isRetryable flag on error responses
- Missing errorCategory on error responses
- Business errors marked as retryable
- Transient errors that trigger immediate escalation instead of retry


## MCP Server Configuration

- **Project-level (.mcp.json):** Version-controlled. Shared with the team. Use for all shared integrations.
- **User-level (~/.claude.json):** Personal. Not version-controlled. Not shared.

Use environment variable expansion (e.g. `${GITHUB_TOKEN}`) to keep credentials out of version control.

Use community MCP servers for standard integrations (Jira, GitHub, Slack). Only build custom servers for workflows that community servers cannot handle.

Flag as violation:
- Shared tool configuration in user-level config instead of project-level
- Credentials hardcoded in .mcp.json instead of using environment variable expansion
- Custom MCP server built for a standard integration without evaluating community alternatives


## Grep vs Glob

- **Grep:** Searches file contents. Use for finding function callers, error messages, import statements.
- **Glob:** Matches file paths. Use for finding files by extension or naming pattern.

For codebase exploration: start with Grep to find entry points, use Read to follow imports. Do not read all files upfront — it kills the context budget.

Flag as violation:
- Using Glob to search for content inside files
- Using Grep to find files by name or extension
- Reading all files in a directory before identifying which ones are relevant