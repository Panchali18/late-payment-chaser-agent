# Data Model (v1)

Four Dataverse tables. Amounts are stored in the invoice currency; there is no currency conversion in v1.

```
Settings (1 row)

Clients 1 ──── * Invoices 1 ──── * Reminders log
```

## Settings

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
| Sender name | Text | Office Manager | |
| Sender role | Text | Accounts | |
| Owner email | Email | test inbox | Escalations go here |
| Signature | Multiline text | | |
| Tone | Choice | Friendly / Neutral / Formal | Default tone for drafts |

## Clients

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

## Invoices

| Column | Type | Notes |
|---|---|---|
| Invoice number | Text | Primary name column, e.g. KL-2026-0142 |
| Client | Lookup → Clients | |
| Amount | Currency (decimal) | Total invoiced |
| Amount paid | Currency (decimal) | For part payments; outstanding = amount − amount paid |
| Currency | Choice | Same options and order as Clients; must match client currency |
| Issue date | Date only | |
| Due date | Date only | |
| Payment status | Choice | Open, Paid, Disputed. Named to avoid clashing with the system Status column |
| Promised pay date | Date only | Blank if none |
| Last reminder stage | Choice | None, Pre-due, Due, Gentle, Formal, Final, Escalated |
| Last reminder date | Date only | Stops the same invoice being chased twice in one day |

## Reminders log

| Column | Type | Notes |
|---|---|---|
| Invoice | Lookup → Invoices | |
| Stage | Choice | Same values as Last reminder stage |
| Draft text | Multiline text | As written by the agent |
| Edited text | Multiline text | As approved by the human, if changed |
| Decision | Choice | Pending, Approved, Edited, Skipped |
| Outcome | Choice | Sent, Skipped, Escalated |
| Escalation reason | Text | Required when Outcome = Escalated |
| Validation passed | Yes/No | Result of the fact check on save |
| Prompt version | Text | e.g. v1.0, matches `prompts/` |
| Created on | Date and time | System column |
