# Flow: SaveDraft

**Type:** agent tool (manual trigger with four text inputs for testing; switched to the agent trigger later)
**Purpose:** the agent's only way to store a reminder. The draft is checked against the invoice record and the business's rules **before** it is saved. A draft that fails is not saved; the agent gets the failed checks and the correct facts so it can rewrite.
**AI involved:** none. This is the deterministic guardrail around the LLM's output.

## Inputs

| Key | Name | Description |
|---|---|---|
| `text` | invoiceId | Unique ID of the invoice the reminder is for |
| `text_1` | stage | Reminder stage from GetDueInvoices (Pre-due, Due, Gentle, Formal, Final) |
| `text_2` | draftText | Full text of the reminder email |
| `text_3` | promptVersion | Version of the agent instructions, e.g. v1.0 |

## Checks

| Check | Fails when |
|---|---|
| invoiceStillOpen | The invoice has been marked Paid since the list was produced |
| hasInvoiceNumber | The draft doesn't contain the exact invoice number |
| hasAmount | The draft doesn't contain the outstanding amount as `<CURRENCY> <amount>`, e.g. `GBP 4,180.00` |
| hasDueDate | The draft doesn't contain the due date as `d MMMM yyyy`, e.g. `19 October 2026` |
| lateFeeRuleRespected | The draft mentions a late fee when the client is excluded or the stage isn't Formal/Final |
| noForbiddenContent | The draft mentions waiving, cancelling, interest, legislation, statutory claims or legal action |

The agent's instructions tell it to write amounts and dates in exactly these formats, so the check is a simple text match.

## Design

| # | Action | Type |
|---|---|---|
| 1 | Manually trigger a flow | Trigger (4 text inputs) |
| 2 | In | Compose: tidies the inputs |
| 3 | GetInvoice | Dataverse: Get a row by ID (Invoices, row ID `outputs('In')?['invoiceId']`) |
| 4 | GetClient | Dataverse: Get a row by ID (Clients, row ID `outputs('GetInvoice')?['body/_lpc_client_value']`) |
| 5 | Expected | Compose: the facts the draft must contain |
| 6 | Checks | Compose: the six checks |
| 7 | Passed | Compose: true only if all checks pass |
| 8 | Condition | `outputs('Passed')` is equal to `true` |
| 9a | AddReminder (yes) | Dataverse: Add a new row (Reminder Logs), Decision Pending, Validation passed Yes |
| 10a | OutputSaved (yes) | Compose |
| 9b | OutputRejected (no) | Compose |

## Expressions

### In
```
addProperty(addProperty(addProperty(addProperty(json('{}'), 'invoiceId', trim(coalesce(triggerBody()?['text'], ''))), 'stage', trim(coalesce(triggerBody()?['text_1'], ''))), 'draftText', coalesce(triggerBody()?['text_2'], '')), 'promptVersion', coalesce(triggerBody()?['text_3'], 'v1.0'))
```

### Expected
```
addProperty(addProperty(addProperty(addProperty(addProperty(json('{}'), 'invoiceNumber', outputs('GetInvoice')?['body/lpc_invoicenumber']), 'currency', outputs('GetInvoice')?['body/lpc_currency@OData.Community.Display.V1.FormattedValue']), 'amount', formatNumber(sub(outputs('GetInvoice')?['body/lpc_amount'], coalesce(outputs('GetInvoice')?['body/lpc_amountpaid'], 0)), 'N2', 'en-US')), 'dueDate', formatDateTime(take(outputs('GetInvoice')?['body/lpc_duedate'], 10), 'd MMMM yyyy')), 'lateFeeAllowed', and(equals(outputs('GetClient')?['body/lpc_latefeeapplies'], true), or(equals(outputs('In')?['stage'], 'Formal'), equals(outputs('In')?['stage'], 'Final'))))
```

### Checks
```
addProperty(addProperty(addProperty(addProperty(addProperty(addProperty(json('{}'), 'invoiceStillOpen', not(equals(outputs('GetInvoice')?['body/lpc_paymentstatus@OData.Community.Display.V1.FormattedValue'], 'Paid'))), 'hasInvoiceNumber', contains(outputs('In')?['draftText'], outputs('Expected')?['invoiceNumber'])), 'hasAmount', contains(outputs('In')?['draftText'], concat(outputs('Expected')?['currency'], ' ', outputs('Expected')?['amount']))), 'hasDueDate', contains(outputs('In')?['draftText'], outputs('Expected')?['dueDate'])), 'lateFeeRuleRespected', or(outputs('Expected')?['lateFeeAllowed'], not(or(contains(toLower(outputs('In')?['draftText']), 'late fee'), contains(toLower(outputs('In')?['draftText']), 'late payment fee'))))), 'noForbiddenContent', not(or(contains(toLower(outputs('In')?['draftText']), 'waive'), contains(toLower(outputs('In')?['draftText']), 'cancel'), contains(toLower(outputs('In')?['draftText']), 'interest'), contains(toLower(outputs('In')?['draftText']), 'statutory'), contains(toLower(outputs('In')?['draftText']), 'legislation'), contains(toLower(outputs('In')?['draftText']), 'legal action'))))
```

### Passed
```
and(outputs('Checks')?['invoiceStillOpen'], outputs('Checks')?['hasInvoiceNumber'], outputs('Checks')?['hasAmount'], outputs('Checks')?['hasDueDate'], outputs('Checks')?['lateFeeRuleRespected'], outputs('Checks')?['noForbiddenContent'])
```

### AddReminder fields
- Reminder reference: `concat(outputs('Expected')?['invoiceNumber'], ' | ', outputs('In')?['stage'], ' | ', convertFromUtc(utcNow(), 'AUS Eastern Standard Time', 'dd/MM/yyyy'))`
- Invoice (lookup): `concat('lpc_invoices(', outputs('In')?['invoiceId'], ')')`
- Stage: `json('{"None":855830000,"Pre-due":855830001,"Due":855830002,"Gentle":855830003,"Formal":855830004,"Final":855830005,"Escalated":855830006}')?[outputs('In')?['stage']]`
- Draft text: `outputs('In')?['draftText']`
- Decision: Pending
- Validation passed: Yes
- Prompt version: `outputs('In')?['promptVersion']`

### OutputSaved
```
addProperty(addProperty(addProperty(json('{}'), 'saved', true), 'reminderId', outputs('AddReminder')?['body/lpc_reminderlogid']), 'message', 'Draft passed validation and is saved for human review. It has not been sent.')
```

### OutputRejected
```
addProperty(addProperty(addProperty(addProperty(json('{}'), 'saved', false), 'failedChecks', outputs('Checks')), 'expectedFacts', outputs('Expected')), 'message', 'Draft rejected. Rewrite it so every check is true, using the expected facts exactly as given, then call SaveDraft again.')
```

## Simulated test (before building)

| Case | Expected | Result |
|---|---|---|
| Correct Formal draft, Northgate | Saved | Pass |
| Wrong amount | Rejected: hasAmount | Pass |
| Late fee for Harbourview (excluded) | Rejected: lateFeeRuleRespected | Pass |
| Draft says "waive" (injection obeyed) | Rejected: noForbiddenContent | Pass |
| Part-paid invoice quoting the balance | Saved | Pass |
| Late fee at Gentle stage | Rejected: lateFeeRuleRespected | Pass |

## Known limitation

Fact checks look for the required facts, not for extra wrong ones. A draft containing the right amount **and** a second, invented amount would pass. The human review step and the evaluation's "invented content" metric cover this in v1.

## Test result (14/10/2026)

- Correct Formal draft for KL-2026-0112: **True** branch, saved to Reminder Log as Pending.
- Same draft with the amount changed to GBP 4,800.00: **False** branch, nothing saved.
- Testing note: Power Automate's test panel can rerun the previous inputs; choose **Manually** and re-enter values for each negative test.
