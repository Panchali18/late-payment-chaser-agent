# Solution

`LatePaymentChaser_1_0_0_1.zip` is the exported Power Platform solution (unmanaged, version 1.0.0.1), publisher prefix `lpc`.

## What's inside

| Component | Items |
|---|---|
| Tables | Business Setting, Client, Invoice, Reminder Log (columns, choices, forms and views) |
| Agent flows | GetDueInvoices, GetClientContext, SaveDraft, EscalateToOwner |
| Agent | Late Payment Chaser: instructions, the four tools and the payment terms knowledge document |

## Not included

- **Reminder Review app.** It was generated with Copilot Studio App (preview) and doesn't export with the solution yet. Its brief and features are in [apps/reminder-review.md](../apps/reminder-review.md).
- **Data.** Synthetic rows are in [data/](../data/) and are imported separately.
- **Credentials.** The Dataverse connection reference is a placeholder; on import you pick or create your own connection. The export was checked for passwords, secrets and tokens before committing.

## Importing

1. In a Power Platform environment with Dataverse, go to **Solutions → Import solution** and select the zip.
2. When asked, choose a Dataverse connection for the connection reference.
3. Import the CSVs from [data/](../data/) (clients first, then invoices) and create one Business Setting row.
4. Open the agent in Copilot Studio. The environment needs Copilot Credits allocated before you can test (see [agent setup](../docs/agent-setup.md)).
