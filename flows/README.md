# Flows

What each Power Automate flow does, its inputs and outputs, and screenshots.

| Flow | Type | Purpose |
|---|---|---|
| GetDueInvoices | Agent tool | Rule-based list of invoices to chase, with stage and reason |
| GetClientContext | Agent tool | Client relationship, notes and past reminders |
| SaveDraft | Agent tool | Validates facts, then saves the draft as Pending |
| EscalateToOwner | Agent tool | Logs an escalation with a reason and notifies the owner |
| SendApprovedReminder | Cloud flow (no AI) | Sends approved drafts to the test inbox and logs the outcome |
