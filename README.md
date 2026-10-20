# Late-Payment Chaser Agent

A human-in-the-loop AI agent that works out which invoices need a follow-up for a small B2B business, drafts the reminders, and flags the cases the owner should handle personally.

> **Demonstration only.** The business, clients, invoices and contacts are fictional and all data is synthetic. No real emails are sent to clients.

**Status:** v1 complete end to end in a sandbox: rule engine, four agent tools, Copilot Studio agent, two evaluation runs, a review app with client context, a rule-based "handle personally" flag and a send-and-log flow. v2 ideas are in the [build log](build-log.md).

---

## The problem

Small B2B businesses lose cash to late invoices because it's hard to keep track of which invoices need a follow-up and which don't: what's coming due, what's overdue, what a client has promised to pay, what's been queried, and who was already reminded. Accounting tools send fixed-schedule reminders that ignore that context: a queried invoice, a promised payment date, a long-standing client who deserves a softer tone, or a client who has already paid but isn't marked as paid.

## What the agent does

1. The office manager asks the agent: *"What needs chasing?"*
2. The agent gets a rule-based list of invoices that are due within 7 days or overdue, each with a reminder stage, an action (draft or escalate) and a reason.
3. For each invoice, the agent either **drafts a reminder** or **escalates to the owner**. The rules escalate known cases (queried, promised date missed, final notice already sent, zero balance still open, client flagged "handle personally"); the agent can also escalate when a client's notes describe a sensitive situation.
4. The office manager reviews each draft in a review app that shows the client's notes and the outstanding amount, edits the text if needed, and clicks **Approve** or **Skip** after confirming the invoice is still unpaid. Escalations appear in a separate **Handle personally** list.
5. A separate, non-AI flow logs each approved reminder as sent and updates the invoice. The sandbox can't send email, so nothing actually goes out.
6. Paid, promised or already-reminded invoices drop out of the next run.

## Fictional business

**Kestrel Lane Print & Signage Pty Ltd**, a small print and signage company in Melbourne, Australia, with B2B clients in Australia, New Zealand, the UK and the US. See the [business profile](docs/business-profile.md).

## Design: rules vs LLM

| Decided by rules (deterministic) | Decided by the LLM |
|---|---|
| Which invoices are chased, and when | The wording of each reminder |
| Reminder stage (from the business's own ladder) | Whether a client's notes call for escalation instead of a draft |
| Known escalations: queried, promised date missed, final notice sent, zero balance, client flagged "handle personally" | |
| Facts in the draft: invoice number, amount, currency, due date | Tone within the limits set by the business |
| Whether a late fee can be mentioned (from the business's own terms) | |

**Guardrails**

- **Least privilege:** the agent has no send tool. Only a non-AI flow, triggered by a human approval, can mark a reminder as sent.
- **Human review with context:** the reviewer sees the client's notes and the outstanding amount next to every draft.
- **Fact validation:** every saved draft is checked against the invoice record. Any mismatch blocks the draft.
- **No legal claims:** late fees are mentioned only if the business's own payment terms include them. The agent never cites law or calculates statutory interest.
- **Untrusted data:** client notes are treated as data, not instructions, to resist prompt injection.
- **B2B only.**

## Tech stack

| Layer | Tool |
|---|---|
| Agent | Microsoft Copilot Studio (generative orchestration, GPT-5 Chat) · [setup](docs/agent-setup.md) |
| Agent tools | Four Power Automate flows converted to Copilot Studio agent flows · [flows/](flows/) |
| Data | Dataverse (Power Apps Developer Plan) |
| Review screen | Reminder Review app, generated from a written brief with Copilot Studio App (preview), then redesigned · [app](apps/reminder-review.md). In production, approvals would arrive in Teams or Outlook |
| Send and log | Non-AI flow triggered by an approval: logs it as sent and updates the invoice · [flow](flows/SendApprovedReminder.md) |
| Knowledge | One-page payment terms document |
| Documentation | GitHub (this repo) |

All on free tiers and trials in a personal sandbox tenant, using a one-month Copilot Studio trial (25,000 Copilot Credits).

## Evaluation

The agent is tested against 30 synthetic invoices covering disputes, part payments, promised dates, new and long-standing clients, four currencies, 14/30/60-day terms, clients excluded from late fees, a vulnerable-customer note, and a prompt-injection attempt in a client note.

| Metric | Target | Run 1 (v1.0) | Run 2 (v1.1) |
|---|---|---|---|
| Correct stage | 100% | 100% | 100% |
| Correct tool path | ≥ 90% | 87.5% | **95.8%** |
| Every listed invoice has a record | 100% | 92% | **100%** |
| Facts correct in saved drafts | 100% | 100% | 100% |
| Unsafe content | 0 | 0 | 0 |
| Invented content | 0 | 2 | **0** |
| Escalation recall | 100% | 86% | 86% |
| Prompt injection held | Yes | Yes | Yes |

Reports: [run 1](eval/runs/run1-report.md) · [run 2](eval/runs/run2-report.md). All rule-enforced checks held in both runs. Prompt v1.1 removed the run 1 errors (confused invoices, a silent miss, invented content). The remaining miss was the LLM's inconsistent judgement on a sensitive client. That decision has since moved into a rule-based "handle personally" flag, and both of that client's invoices now escalate every time (run 3 to confirm end to end).

## Deploying for a real business

The design keeps business specifics in data, so adapting it means changing the data layer, not the agent.

| Area | Demo | Real business |
|---|---|---|
| Invoices and clients | Synthetic data imported from Excel | Synced from the accounting system (for example Xero, QuickBooks or MYOB via Power Automate connectors) or imported on a schedule |
| Business rules | Kestrel Lane's settings row | The business's own terms, reminder ladder, signature and tone, edited in one row |
| Run date | Fixed (03/11/2026) for repeatable tests | Blank, so the real date is used |
| Sending | Logged only | Approved drafts sent from the business's mailbox |
| Running | On request | On request or a scheduled daily run |
| Licensing | Free trials | Copilot Studio credits and Power Apps licences |
| Go-live | — | Data protection review, a pilot with human approval of every draft, and the same evaluation run on the business's own data |

## Repository layout

```
docs/          project brief, business profile, user stories, data model
data/          synthetic CSVs to import into Dataverse
knowledge/     payment terms document used by the agent
prompts/       agent instructions and tool descriptions, versioned
apps/          the review app, the brief used to generate it, and the Teams/Outlook approval card design
flows/         what each Power Automate flow does, with its expressions
solution/      exported Power Platform solution (taken before the final changes; see its README)
eval/          test set and results
build-log.md   each build step and what I learned
```

## Limitations (v1)

- No real email sending to clients, payment integration or bank feeds.
- No interest calculation and no B2C chasing.
- Runs on request only, not on a schedule.
- English drafts only.
- The agent and review app run in a personal sandbox tenant, so they can't be opened by others; the [build log](build-log.md) and [app doc](apps/reminder-review.md) show how they work.

## What's next (v2)

- One combined reminder per client, instead of one per invoice.
- A warning on the review screen when a client note contains instruction-like text.
- Approvals in Teams or Outlook using the [approval card](apps/teams-approval-card.json), and real sending from a shared mailbox.
- Evaluation run 3 with the "handle personally" rule in place.

## Author

Built by **Panchali Das** — [LinkedIn](https://www.linkedin.com/in/panchalidas/)

## Licence

[MIT](LICENSE)
