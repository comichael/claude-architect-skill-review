## Lost in the Middle

Models process the beginning and end of long inputs reliably. Information buried in the middle gets missed.

Place key findings and summaries at the beginning of long inputs. If passing results from multiple sources, put the most important results first.

Flag as violation:
- Key findings or critical data placed only in the middle of a long prompt
- Multi-source input with no summary or priority ordering at the top


## Escalation Triggers

Three valid triggers for escalating to a human:
- **Customer explicitly requests a human.** Honor immediately. Do not attempt to resolve first.
- **Policy exceptions or gaps.** The request falls outside documented policy.
- **No meaningful progress** after multiple attempts.

Two unreliable triggers (common distractors):
- **Sentiment-based escalation.** A frustrated customer with a simple question needs their answer, not a human.
- **Self-reported confidence scores.** The model is often incorrectly confident on hard cases and uncertain on easy ones.

Flag as violation:
- Escalation logic based on detected frustration or sentiment alone
- Escalation logic based on model self-reported confidence
- Agent that attempts to resolve after customer explicitly asks for a human


## Error Propagation

Two anti-patterns:
- **Silent suppression:** Subagent fails, returns empty results marked as success. No recovery possible.
- **Workflow termination:** Single failure kills the entire pipeline. All partial results lost.

Correct pattern: structured error context. A failing subagent reports: failure type, what was attempted, partial results gathered, and potential alternatives. The coordinator decides whether to retry, reroute, or proceed with gaps annotated.

Flag as violation:
- Subagent that returns empty results with no error indicator on failure
- Pipeline that terminates entirely on a single subagent failure
- Error propagation with no partial results or description of what was attempted
- Coordinator with no logic to handle partial failures


## Information Provenance

Each finding in a multi-agent synthesis needs: claim, source URL, document name, relevant excerpt, and publication date.

When two sources report different values, do not pick one. Annotate both with their sources and dates. Different dates often explain different numbers — they are not contradictions.

Flag as violation:
- Synthesis output with claims that have no source attribution
- Conflicting data resolved by silently picking one value
- Missing publication dates on sources where temporal context matters


## Independent Review

The same Claude instance that produced output is less effective at reviewing it. It retains reasoning context that biases it toward its own decisions.

Use a separate instance with no shared conversation history for quality review. Pass only the output and the requirements — not the reasoning that produced it.

Flag as violation:
- Code review performed by the same session that wrote the code
- Quality check that includes the original reasoning chain alongside the output
- No independent review step for high-stakes output


## Codebase Exploration

Extended sessions degrade context. The model starts referencing "typical patterns" instead of specific classes it discovered earlier.

Mitigations:
- Write key findings to a scratchpad file and reference it in subsequent prompts
- Delegate specific investigations to subagents, keep the main agent at coordination level
- Summarize findings from one phase before starting the next
- Use /compact to reduce context when it fills with verbose discovery output

Flag as violation:
- Long exploration session with no scratchpad or summary checkpoints
- Main agent performing both deep investigation and high-level coordination without delegation