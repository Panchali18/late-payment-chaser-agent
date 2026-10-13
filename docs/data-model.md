# Data Model (v1)

Four Dataverse tables in the **Late Payment Chaser** solution (publisher prefix `lpc_`). Amounts are stored in the invoice currency as decimals; there is no currency conversion in v1. All date columns are date only and time-zone independent, so due dates never shift between time zones.

```
Business Setting (1 row)

Client 1 ──── * Invoice 1 ──── * Reminder Log
```

## Business Setting (`lpc_BusinessSetting`)

One row holding the business's configuration. Changing this table is how the agent fits a different small business.

| Column | Type | Example | Notes |
|---|---|---|---|
| Business name | Text | Kestrel Lane Print & Signage Pty Ltd | |
| Country | Text | Australia | |
| Home currency | Text | AUD | |
| Default terms (days) | Whole number | 30 | |
| Ladder: pre-due offset | Whole number | -7 | Days relative to due date |
| Ladder: gentle offset | Whole number | 3 | |
| Ladder: formal offset | Whole number | 14 | |
| Ladder: final offset | Whole number | 30 | |
| Sender name | Text | Jordan Hale (fictional) | |
| Sender role | Text | Accounts Manager | Optional |
| Owner email | Email | owner@kestrellane.example | Escalations go here; switched to a test inbox for sending |
| Signature | Multiline text | | |
| Tone | Choice | Friendly / Neutral / Formal | Default tone for drafts |
| Run date override | Date only | 03/11/2026 | If set, treated as today so test runs are repeatable; blank in normal use |

## Client (`lpc_Client`)

| Column | Type | Notes |
|---|---|---|
| Client name | Text | Primary name column |
| Billing contact name | Text | |
| Billing contact email | Email | Test addresses only |
| Country | Text | AU, NZ, UK or US |
| Currency | Choice | AUD, NZD, GBP, USD, EUR, INR |
| Relationship | Choice | New, Regular, Long-standing |
| Payment terms override (days) | Whole number | Blank means use the default |
| Late fee applies | Yes/No | No for clients excluded from the late-fee clause |
| Notes | Multiline text | **Untrusted data.** May contain injection attempts in tests |

## Invoice (`lpc_Invoice`)

| Column | Type | Notes |
|---|---|---|
| Invoice number | Text | Primary name column, e.g. KL-2026-0142 |
| Client | Lookup → Clients | |
| Amount | Decimal (2 places, min 0) | Total invoiced, including tax. Decimal rather than Dataverse's Currency type, to avoid exchange-rate columns |
| Amount paid | Decimal (2 places, min 0) | For part payments; outstanding = amount − amount paid |
| Currency | Choice | Same options and order as Clients; must match client currency |
| Issue date | Date only | |
| Due date | Date only | |
| Payment status | Choice | Open, Paid, Queried (client has raised a query or dispute). Named to avoid clashing with the system Status column |
| Promised pay date | Date only | Blank if none |
| Last reminder stage | Choice | None, Pre-due, Due, Gentle, Formal, Final, Escalated |
| Last reminder date | Date only | Stops the same invoice being chased twice in one day |

## Reminder Log (`lpc_ReminderLog`)

| Column | Type | Notes |
|---|---|---|
| Reminder reference | Text | Primary name column, filled by the flow, e.g. KL-2026-0142 \| Gentle \| 13/10/2026 |
| Invoice | Lookup → Invoice | |
| Stage | Choice | Same options, order and values as Last reminder stage |
| Draft text | Multiline text (4,000 chars) | As written by the agent; blank for escalations |
| Edited text | Multiline text (4,000 chars) | As approved by the human, if changed |
| Decision | Choice | Pending (default), Approved, Edited, Skipped |
| Outcome | Choice | Sent, Skipped, Escalated; blank until review is complete |
| Escalation reason | Text | Required when Outcome = Escalated |
| Validation passed | Yes/No | Result of the fact check on save; default No |
| Prompt version | Text | e.g. v1.0, matches `prompts/` |
| Created on | Date and time | System column |
