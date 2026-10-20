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
| Owner email | Email | owner@kestrellane.example | Not used in v1: escalations appear in the review app, not by email |
| Signature | Multiline text | | |
| Tone | Choice | Friendly / Neutral / Formal | Default tone for drafts |
| Run date override | Date only | 03/11/2026 | If set, treated as today so test runs are repeatable; blank in normal use |

## Client (`lpc_Client`)

| Column | Type | Notes |
|---|---|---|
| Client name | Text | Primary name column |
| Billing contact name | Text | |
| Billing contact email | Email | Test addresses only |
| Country | Text | Australia, New Zealand, United Kingdom or United States (the Currency formula column relies on these exact values) |
| Currency | Choice | AUD, NZD, GBP, USD, EUR, INR |
| Relationship | Choice | New, Regular, Long-standing |
| Payment terms override (days) | Whole number | Blank means use the default |
| Late fee applies | Yes/No | No for clients excluded from the late-fee clause |
| Notes | Multiline text | **Untrusted data.** May contain injection attempts in tests |
| Handle personally | Yes/No (default No) | Added after evaluation run 2. When Yes, GetDueInvoices escalates every invoice for this client, so sensitive clients never get an automatic draft |

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
| Edited text | Multiline text (4,000 chars) | The text the reviewer approved (the app saves it on every approval), or the owner's "What did you do?" note for escalations |
| Decision | Choice | Pending (default), Approved, Edited, Skipped. "Mark as handled" on an escalation sets Skipped |
| Outcome | Choice | Sent, Skipped, Escalated; blank until review is complete |
| Escalation reason | Text | Required when Outcome = Escalated |
| Validation passed | Yes/No | Result of the fact check on save; default No |
| Prompt version | Text | e.g. v1.0, matches `prompts/` |
| Created on | Date and time | System column |

**Formula columns (added for the review app).** Read-only, calculated by Dataverse through the Invoice lookup, so existing rows fill in with no flow changes:

| Column | Formula | Notes |
|---|---|---|
| Client name | `Invoice.'Client '.'Client name'` | |
| Outstanding | `Invoice.'Amount ' - Invoice.'Amount paid'` | First built as the invoice total, which contradicted correct drafts for part-paid invoices; changed to the outstanding balance |
| Due date | `Invoice.'Due date'` | |
| Contact name | `Invoice.'Client '.'Billing contact name'` | |
| Client notes | `Invoice.'Client '.Notes` | Lets the reviewer see sensitive-client notes |
| Currency | `Switch(Invoice.'Client '.Country, "Australia","AUD", "New Zealand","NZD", "United Kingdom","GBP", "United States","USD", "")` | Demo shortcut, see below |

Some display names carry a trailing space (`'Client '`, `'Amount '`), so formulas are built by picking from the editor's suggestion list rather than typing.

**No contact email:** the billing email is an Email-type column, which formula columns couldn't read. The reviewer only needs the name.

**Currency shortcut:** formula columns can't read choice columns (not through a lookup, and `Text()` on a same-table choice also failed), so currency is derived from the client's country. This holds for the demo data, where every client is billed in its home currency. A real business billing one client in several currencies would store the currency code as plain text on the invoice.
