# Flow: SendApprovedReminder

**Type:** automated Power Automate cloud flow. Not an agent tool: the agent has no way to send (least privilege).
**Purpose:** close the loop after human review. When a reminder is approved, log it as sent and update the invoice so it drops out of the next run.
**AI involved:** none.

## Design

| # | Action | Type | What it does |
|---|---|---|---|
| 1 | When a row is added, modified or deleted | Dataverse trigger | Change type Modified, table Reminder logs, scope Organization |
| 2 | LogSent | Dataverse: Update a row | Reminder log: Outcome = Sent |
| 3 | UpdateInvoice | Dataverse: Update a row | Invoice: Last reminder stage = the reminder's stage, Last reminder date = today |

## Trigger settings

- **Select columns:** `lpc_decision`, so the flow only wakes when the Decision changes. Its own update to Outcome doesn't retrigger it.
- **Filter rows:** `(lpc_decision eq 855830001 or lpc_decision eq 855830002) and lpc_outcome eq null`. Approved (855830001) or Edited (855830002), and not already logged, so a reminder can't be "sent" twice.

## Expressions

- UpdateInvoice → Row ID: `Invoice (Value)` from the trigger (`_lpc_invoice_value`)
- Last reminder stage: `triggerOutputs()?['body/lpc_stage']` (Reminder log Stage and Invoice Last reminder stage share the same choice values)
- Last reminder date: `utcNow('yyyy-MM-dd')`

## Sending

The sandbox tenant has no Exchange licence, so v1 logs the reminder as sent rather than emailing it. In production, a "Send an email (V2)" step from the business's shared mailbox goes before LogSent, using Edited text if present, otherwise Draft text, and the client's billing email.

## Test result (17/10/2026)

Approved KL-2026-0127 in the Reminder Review app: the flow ran within a minute and succeeded (1 second).
