## Explicit Criteria

Vague instructions like "be conservative" or "only report high-confidence findings" produce inconsistent results. Replace with specific categorical criteria.

Wrong: "Be conservative when flagging code issues."
Right: "Flag only: null references, index out of range, SQL injection, missing authentication. Do not flag: naming conventions, missing comments, code style preferences."

Severity levels must be defined with concrete examples, not prose descriptions of what each level means.

High false positive rates in one category destroy trust in all categories. Fix: temporarily disable the high-false-positive category, improve its prompts, then re-enable.

Flag as violation:
- System prompt containing "be conservative", "be careful", "only if confident" or similar vague guidance
- Severity levels defined without concrete examples
- No explicit list of what to flag and what to skip


## Few-Shot Prompting

The most effective technique for consistent output. Not more instructions. Not confidence thresholds. Examples.

Use 2-4 targeted examples for ambiguous scenarios. Each example must show the reasoning for why one action was chosen over alternatives. This teaches generalization, not rigid pattern matching.

Deploy when:
- Detailed instructions alone produce inconsistent formatting
- Model makes inconsistent judgment calls on edge cases
- Extraction tasks return empty fields for information that exists in the source

Flag as violation:
- Inconsistent output addressed by adding more prose instructions instead of examples
- Few-shot examples that show only the answer without reasoning
- More than 6 examples (diminishing returns, wasted tokens)


## Structured Output with tool_use

tool_use with JSON schemas eliminates syntax errors. It does not prevent:
- **Semantic errors:** Line items that do not sum to stated total.
- **Field placement:** Values in wrong fields.
- **Fabrication:** Model invents values for required fields when source lacks the information.

Schema design rules:
- **Nullable fields** for data the source may not contain. Prevents fabrication.
- **"unclear" enum value** for genuinely ambiguous cases.
- **"other" + freeform detail string** for extensible categorization.
- All fields that are not guaranteed to exist in every source document should be optional or nullable.

Flag as violation:
- All fields marked as required when some may not exist in the source
- No nullable fields in an extraction schema
- No "unclear" or equivalent option for ambiguous data
- Schema with enum that has no catch-all option for unexpected values


## Validation-Retry Loops

When extraction fails validation, send back: original document, failed extraction, and the specific validation error. The model uses the error to self-correct.

Effective for: format mismatches, structural errors, misplaced values.
Not effective for: information that does not exist in the source document.

Do not burn retry cycles on missing data. If the source does not contain the information, no amount of retries will produce it.

Flag as violation:
- Retry loop with no error feedback (just re-running the same extraction)
- Retrying more than twice on the same validation error
- Retrying on fields where the source document genuinely lacks the information


## Persistent Case Facts

Transactional data (amounts, dates, IDs, order numbers) must be stored in a separate structured object and included in every API call. Never rely on conversation history to preserve these values.

Progressive summarization compresses critical details. "$247.83 for order #8891 from March 3rd" becomes "the refund" over a long conversation.

Fix: extract facts into a dedicated object in your code and pass it in the system prompt on every call.

Flag as violation:
- Transactional data stored only in conversation history with no separate facts object
- System prompt that does not include persistent case facts when the conversation involves specific amounts, dates or IDs
- Facts object that gets summarized or paraphrased instead of passed verbatim


## Batch Processing

Message Batches API: 50% cost savings, up to 24-hour processing window, no latency guarantee, no multi-turn tool calling within a single request.

- **Synchronous API:** Blocking workflows. Pre-merge checks, anything a developer waits for.
- **Batch API:** Latency-tolerant workflows. Overnight reports, weekly audits, nightly test generation.

Refine prompts on a small sample before submitting large batch volumes.

Flag as violation:
- Blocking developer workflow using Batch API instead of synchronous
- Large batch submitted without prior prompt refinement on a sample set
- Batch processing used for tasks that require multi-turn tool calling