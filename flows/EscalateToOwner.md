# Flow: EscalateToOwner

**Type:** agent tool (manual trigger with four text inputs for testing; switched to the agent trigger later)
**Purpose:** record that an invoice needs the owner's personal attention, with the reason. No reminder is drafted.
**AI involved:** none.

## Inputs

| Key | Name | Description |
|---|---|---|
| `text` | invoiceId | Unique ID of the invoice being escalated |
| `text_1` | stage | Stage from GetDueInvoices (use Escalated if the final notice has already been sent) |
| `text_2` | reason | Why the owner should handle this personally, in one sentence |
| `text_3` | promptVersion | Version of the agent instructions, e.g. v1.0 |

## Design

| # | Action | Type |
|---|---|---|
| 1 | Manually trigger a flow | Trigger (4 text inputs) |
| 2 | In | Compose |
| 3 | GetInvoice | Dataverse: Get a row by ID (Invoices) |
| 4 | AddEscalation | Dataverse: Add a new row (Reminder Logs) |
| 5 | Output | Compose |

## Expressions

### In
```
addProperty(addProperty(addProperty(addProperty(json('{}'), 'invoiceId', trim(coalesce(triggerBody()?['text'], ''))), 'stage', trim(coalesce(triggerBody()?['text_1'], ''))), 'reason', coalesce(triggerBody()?['text_2'], '')), 'promptVersion', coalesce(triggerBody()?['text_3'], 'v1.0'))
```

### AddEscalation fields
- Reminder reference: `concat(outputs('GetInvoice')?['body/lpc_invoicenumber'], ' | Escalated | ', convertFromUtc(utcNow(), 'AUS Eastern Standard Time', 'dd/MM/yyyy'))`
- Invoice (lookup): `concat('lpc_invoices(', outputs('In')?['invoiceId'], ')')`
- Stage: `json('{"None":855830000,"Pre-due":855830001,"Due":855830002,"Gentle":855830003,"Formal":855830004,"Final":855830005,"Escalated":855830006}')?[outputs('In')?['stage']]`
- Decision: Pending (so it appears in the owner's review list)
- Validation passed: No (there is no draft to validate)
- Outcome: Escalated
- Escalation reason: `outputs('In')?['reason']`
- Prompt version: `outputs('In')?['promptVersion']`

### Output
```
addProperty(addProperty(addProperty(json('{}'), 'escalated', true), 'reminderId', outputs('AddEscalation')?['body/lpc_reminderlogid']), 'message', 'Escalated to the owner with the reason given. No reminder has been drafted or sent for this invoice.')
```

## v1 note

The owner sees escalations in the review screen (Decision = Pending, Outcome = Escalated). An email notification is added with the send-and-log flow.
