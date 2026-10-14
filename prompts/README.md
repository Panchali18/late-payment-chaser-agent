# Prompts

Agent instructions, versioned. The version (for example `v1.0`) is passed to SaveDraft and EscalateToOwner and stored on every Reminder Log row, so evaluation results can be traced to the exact instructions that produced them.

| Version | File | Notes |
|---|---|---|
| v1.0 | [agent-instructions-v1.0.md](agent-instructions-v1.0.md) | First version: tool order, fact formats matching SaveDraft's checks, tone by stage, late-fee rule, injection handling |
| v1.1 | [agent-instructions-v1.1.md](agent-instructions-v1.1.md) | After run 1: one invoice at a time with identifiers restated; escalation reason fixed text (no cross-client leakage); completeness check; summary built only from confirmed tool responses with "Not completed" status; no unsupported compliments; Formal stage wording tightened; 60–120 words |
