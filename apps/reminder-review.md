# Reminder Review app

The human-in-the-loop step: nothing the agent drafts goes anywhere until a person approves it here.

**Built with:** Copilot Studio App (preview), generated from the written brief below. It uses the existing Reminder Log table; no new tables, and no email sending.

## What it does

- **Review queue:** Reminder Log rows with Decision = Pending, newest first. Escalations are marked "Escalated – owner to handle" with the reason.
- **Review detail:** reference, invoice, stage, prompt version, the agent's draft (read-only) and an editable copy.
- **Actions:** Approve, Approve with edits, Skip. Both approvals first ask "Have you confirmed this invoice is still unpaid?" (Yes/Cancel).
- **Tested:** loads live Reminder Log data; Approve saves Decision = Approved and removes the item from the queue.

## Brief used to generate it

```
Build a simple review app for an office manager, using the existing Dataverse table "Reminder Log" (lpc_reminderlog) in this environment. Do not create new tables.

Screen 1 – Review queue: a list of Reminder Log rows where Decision is Pending, newest first. Show Reminder reference, Stage, Validation passed and Escalation reason. Rows with an Escalation reason should be visually marked as "Escalated – owner to handle".

Screen 2 – Review detail: for the selected row, show Reminder reference, the related Invoice, Stage, Draft text (read-only), Escalation reason and Prompt version. Include an editable "Edited text" box pre-filled with Draft text.

Buttons on Screen 2:
- Approve: set Decision to Approved and save.
- Approve with edits: save Edited text, set Decision to Edited and save.
- Skip: set Decision to Skipped and save.
After any button, return to Screen 1.

Before Approve or Approve with edits, show a confirmation: "Have you confirmed this invoice is still unpaid?" with Yes and Cancel.

Do not send any email from the app.
```

## Gap found in testing

The detail screen doesn't show the client's notes, so a reviewer approved a reminder to the hardship client that the agent should have escalated. **Next version:** show client notes and a "sensitive client" warning on the detail screen.

## Still to verify

The Skip and Approve-with-edits paths (Approve is verified).
