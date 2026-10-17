# Flow: GetDueInvoices

**Type:** agent tool. Built and tested with a manual trigger, then switched to **When an agent calls the flow** (same inputs, same order) with a **Respond to the agent** step, and converted to a Copilot Studio agent flow (Plan: Copilot Studio).
**Purpose:** apply the deterministic [chasing rules](../docs/chasing-rules.md) and return the invoices that need action, each with a stage, an action (Draft or Escalate) and a reason.
**AI involved:** none. This flow is pure rules, so its output can be tested exactly against [`eval/test-set.csv`](../eval/test-set.csv).

## Design

No loops: the flow uses `Select` and `Filter array` to transform all invoices in one pass, which is faster and easier to read than an "Apply to each" with nested conditions.

| # | Action | Type | What it does |
|---|---|---|---|
| 1 | When an agent calls the flow | Trigger | Starts the flow (replaced by the agent trigger later) |
| 2 | GetSettings | Dataverse: List rows | Reads the Business Setting row (row count 1) |
| 3 | Settings | Compose | Picks that one row out of the list |
| 4 | RunDate | Compose | Run date override if set, otherwise today in Melbourne |
| 5 | StageRank | Compose | Orders the stages so "later than last reminder" can be checked |
| 6 | GetInvoices | Dataverse: List rows | Reads all invoices |
| 7 | Base | Select | Tidies each invoice and calculates outstanding amount and days overdue |
| 8 | Staged | Select | Adds the stage from the reminder ladder |
| 9 | Listed | Filter array | Keeps only invoices that should be listed |
| 10 | Results | Select | Adds final stage, action and reason |
| 11 | GetFlaggedClients | Dataverse: List rows | Clients with Handle personally = Yes (filter `lpc_handlepersonally eq true`, select `lpc_clientid`) |
| 12 | FlaggedIds | Select | List of flagged client IDs: `item()?['lpc_clientid']` |
| 13 | Final | Select | Any invoice for a flagged client becomes Escalate (see below) |
| 14 | Output | Compose | `body('Final')`, returned to the agent |

## Expressions

Dates are compared as `yyyy-MM-dd` text (`take(..., 10)`), so time zones can't shift them. `take` is used instead of `substring` because it doesn't fail on empty values.

### 3. Settings (Compose)
```
first(outputs('GetSettings')?['body/value'])
```

### 4. RunDate (Compose)
```
if(empty(outputs('Settings')?['lpc_rundateoverride']), convertFromUtc(utcNow(), 'AUS Eastern Standard Time', 'yyyy-MM-dd'), take(coalesce(outputs('Settings')?['lpc_rundateoverride'], ''), 10))
```

### 5. StageRank (Compose)
```
json('{"Not yet":-1,"None":0,"Pre-due":1,"Due":2,"Gentle":3,"Formal":4,"Final":5,"Escalated":6}')
```

### 7. Base (Select)
From: `outputs('GetInvoices')?['body/value']` · Map (text mode):
```
addProperty(addProperty(addProperty(addProperty(addProperty(addProperty(addProperty(addProperty(addProperty(addProperty(addProperty(addProperty(json('{}'), 'invoiceId', item()?['lpc_invoiceid']), 'invoiceNumber', item()?['lpc_invoicenumber']), 'clientId', item()?['_lpc_client_value']), 'clientName', item()?['_lpc_client_value@OData.Community.Display.V1.FormattedValue']), 'currency', item()?['lpc_currency@OData.Community.Display.V1.FormattedValue']), 'outstanding', sub(item()?['lpc_amount'], coalesce(item()?['lpc_amountpaid'], 0))), 'dueDate', take(item()?['lpc_duedate'], 10)), 'daysOverdue', div(sub(ticks(outputs('RunDate')), ticks(take(item()?['lpc_duedate'], 10))), 864000000000)), 'paymentStatus', item()?['lpc_paymentstatus@OData.Community.Display.V1.FormattedValue']), 'promisedDate', take(coalesce(item()?['lpc_promisedpaydate'], ''), 10)), 'lastStage', coalesce(item()?['lpc_lastreminderstage@OData.Community.Display.V1.FormattedValue'], 'None')), 'lastDate', take(coalesce(item()?['lpc_lastreminderdate'], ''), 10))
```

### 8. Staged (Select)
From: `body('Base')` · Map (text mode):
```
addProperty(item(), 'calcStage', if(less(item()?['daysOverdue'], outputs('Settings')?['lpc_ladderpredueoffset']), 'Not yet', if(less(item()?['daysOverdue'], 0), 'Pre-due', if(less(item()?['daysOverdue'], outputs('Settings')?['lpc_laddergentleoffset']), 'Due', if(less(item()?['daysOverdue'], outputs('Settings')?['lpc_ladderformaloffset']), 'Gentle', if(less(item()?['daysOverdue'], outputs('Settings')?['lpc_ladderfinaloffset']), 'Formal', 'Final'))))))
```

### 9. Listed (Filter array)
From: `body('Staged')` · Condition: the expression below **is equal to** `true`
```
and(not(equals(item()?['paymentStatus'], 'Paid')), not(equals(item()?['calcStage'], 'Not yet')), less(ticks(if(empty(item()?['promisedDate']), '1900-01-01', item()?['promisedDate'])), ticks(outputs('RunDate'))), not(equals(item()?['lastDate'], outputs('RunDate'))), or(equals(item()?['lastStage'], 'Final'), greater(outputs('StageRank')?[item()?['calcStage']], outputs('StageRank')?[item()?['lastStage']])))
```

### 10. Results (Select)
From: `body('Listed')` · Map (text mode):
```
addProperty(addProperty(addProperty(item(), 'stage', if(equals(item()?['lastStage'], 'Final'), 'Escalated', item()?['calcStage'])), 'action', if(or(equals(item()?['lastStage'], 'Final'), equals(item()?['paymentStatus'], 'Queried'), not(empty(item()?['promisedDate'])), lessOrEquals(item()?['outstanding'], 0)), 'Escalate', 'Draft')), 'reason', if(equals(item()?['lastStage'], 'Final'), 'Final notice already sent', if(equals(item()?['paymentStatus'], 'Queried'), 'Invoice queried by client', if(not(empty(item()?['promisedDate'])), 'Promised pay date missed', if(lessOrEquals(item()?['outstanding'], 0), 'Balance is zero but invoice is still open; check whether it has been paid', if(less(item()?['daysOverdue'], 0), concat('Due in ', string(mul(item()?['daysOverdue'], -1)), ' days'), if(equals(item()?['daysOverdue'], 0), 'Due today', concat(string(item()?['daysOverdue']), ' days overdue'))))))))
```

### 11. Output (Compose)
```
body('Results')
```

## Expected result on run date 03/11/2026

24 invoices listed: 19 marked Draft by the rules (2 of which the agent should escalate on reading the client notes) and 5 marked Escalate, with stages as in `eval/test-set.csv`. 6 invoices are not listed.

## Test result (14/10/2026)

Run against the 30 test invoices with run date 03/11/2026: **30/30 match the expected outcome**. Full comparison: [`eval/results-getdueinvoices.csv`](../eval/results-getdueinvoices.csv).


## Response to the agent

Respond to the agent, one Text output: `invoices` = `string(outputs('Output'))`.


## Handle personally rule (added 17/10/2026)

Added after evaluation run 2, where the agent drafted a reminder for a client whose notes said "handle personally". The decision moved from the LLM to a rule: a yes/no **Handle personally** column on Client, checked here. Added as three new steps after Results, so none of the tested expressions changed.

### 13. Final (Select)
From: `body('Results')` · Map (text mode):
```
setProperty(setProperty(item(), 'action', if(contains(body('FlaggedIds'), item()?['clientId']), 'Escalate', item()?['action'])), 'reason', if(and(contains(body('FlaggedIds'), item()?['clientId']), equals(item()?['action'], 'Draft')), 'Client flagged: owner handles personally', item()?['reason']))
```
`setProperty` rather than `addProperty`, because the properties already exist. An invoice already escalated for another reason (for example queried) keeps its original reason.

### Test result (17/10/2026)

Wattle & Pine Cafes flagged. Both its invoices (KL-2026-0125 and KL-2026-0126) returned as Escalate with "Client flagged: owner handles personally". 23 invoices listed (16 Draft, 7 Escalate): KL-2026-0127 dropped out because SendApprovedReminder had recorded its Due reminder, which is the intended behaviour.
