# Flows

What each Power Automate flow does, its inputs and outputs, and screenshots.

| Flow | Type | Purpose |
|---|---|---|
| GetDueInvoices | Agent tool | Rule-based list of invoices to chase, with stage and reason |
| GetClientContext | Agent tool | Client relationship, notes and past reminders |
| SaveDraft | Agent tool | Validates facts, then saves the draft as Pending |
| EscalateToOwner | Agent tool | Logs an escalation with a reason and notifies the owner |
| [SendApprovedReminder](SendApprovedReminder.md) | Automated cloud flow (no AI) | When a reminder is approved, logs it as sent and updates the invoice so it isn't chased again |

The four agent tools were built and tested in Power Automate with a manual trigger, then switched to the agent trigger and converted to Copilot Studio agent flows, because the new Copilot Studio only accepts agent flows as tools.
