# Flows

What each Power Automate flow does, its inputs and outputs, and screenshots.

| Flow | Type | Purpose |
|---|---|---|
| GetDueInvoices | Agent tool | Rule-based list of invoices to chase, with stage and reason |
| GetClientContext | Agent tool | Client relationship, notes and past reminders |
| SaveDraft | Agent tool | Validates facts, then saves the draft as Pending |
| EscalateToOwner | Agent tool | Logs an escalation with a reason and notifies the owner |
| SendApprovedReminder | Cloud flow (no AI) | **Planned:** marks approved drafts as sent and logs the outcome |

The four agent tools were built and tested in Power Automate with a manual trigger, then switched to the agent trigger and converted to Copilot Studio agent flows, because the new Copilot Studio only accepts agent flows as tools.
