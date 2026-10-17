# Agent setup (Copilot Studio)

| Setting | Value |
|---|---|
| Name | Late Payment Chaser |
| Environment | Developer environment with Dataverse (personal sandbox tenant) |
| Model | GPT-5 Chat (default) |
| Instructions | [v1.1](../prompts/agent-instructions-v1.1.md) (v1.0 used in run 1) |
| Tools | GetDueInvoices, GetClientContext, SaveDraft, EscalateToOwner (agent flows) |
| Knowledge | Kestrel Lane payment terms (Word document); web search removed |
| Channels | None. Tested in the Preview chat; the trial doesn't allow publishing |
| Memory | Off |

## Licensing and capacity: what it took to run

1. **Copilot Studio trial** (self-service sign-up): unlocks the editor for the user.
2. **Microsoft Copilot Studio Trial** (Microsoft 365 admin centre → Marketplace, EUR 0.00, 1 month): adds 25,000 Copilot Credits to the tenant.
3. **Allocate capacity** (Power Platform admin centre → Licensing → Copilot Studio → Manage capacity): assign credits to the environment, leaving pay-as-you-go off.

Without step 3, the agent stops at its first message: "This environment is out of credits."

**Usage so far:** after two full evaluation runs, the Power Platform admin centre still reports 0 credits consumed for the environment (25,000 allocated). Recorded as "not yet reported by the admin centre", not as "free".

## How to run

In the agent's **Preview** chat, ask "What needs chasing?". The agent works through the list from GetDueInvoices, saves drafts or escalates, and ends with a summary. Results are checked against the Reminder Log, not the chat.
