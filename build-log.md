# Build Log

Each step of the build and what I learned. Newest entries at the bottom.

---

## 13/10/2026 — Environment and repo setup

**Done**
- Created a free Azure account with a personal Microsoft account. This also created a Microsoft Entra tenant.
- Created a separate tenant user for building, because the Power Apps Developer Plan and Copilot Studio trial need a work or school account, not a personal one.
- Set the user's usage location, which licences and trials need.
- Signed in as that user for the first time and set up two-step verification (SMS, after the Authenticator QR setup didn't work).
- Created this GitHub repo with README, MIT licence and starter docs, and gave the Claude GitHub App access to it so docs can be committed directly.
- Signed up for the Power Apps Developer Plan as the builder user, which created a free developer environment with Dataverse.
- Created a solution, **Late Payment Chaser**, with my own publisher (prefix `lpc`), so every table, flow and app sits in one exportable package.
- Started the first table, **Business Setting**, with descriptions on the table and primary column.

**Learned**
- A personal Microsoft account can't sign up for Power Platform developer tools directly. An Azure free account is a free way to get an Entra tenant and a work-style account.
- The card at Azure sign-up is for identity verification; the spending limit stops charges unless you upgrade to pay-as-you-go.
- Keep personal, employer and sandbox sign-ins in separate browser profiles to avoid session mix-ups.
- Building inside a solution from the start matters: it keeps the project portable and gives a single file to version in GitHub.
- The publisher prefix must start with a letter; it is added to every table and column name (for example `lpc_invoice`).
- Table and column descriptions are worth writing early, because the agent can use them to understand the data.
- A connected GitHub account isn't enough for Claude to commit; the Claude GitHub App also needs access to the specific repo.

---

## 13/10/2026 — Data model built in Dataverse

**Done**
- Built all four tables in the solution: Business Setting, Client, Invoice and Reminder Log, with descriptions on every table and column.
- Added the business's settings row for Kestrel Lane (terms, reminder ladder, sender, signature, tone).
- Used choice columns for currency, relationship, payment status, reminder stage, decision and outcome, with matching option values where two tables share a list.
- Added lookups: Invoice → Client, and Reminder Log → Invoice.

**Decisions**
- Amounts are **Decimal**, not Dataverse's Currency type, which would add exchange-rate columns the agent doesn't need.
- Date columns are **time-zone independent**, so a Melbourne due date never shifts when viewed from Dublin.
- The invoice status column is **Payment status**, to avoid clashing with the built-in Status column, and the dispute state is labelled **Queried**.
- Added EUR and INR as currency options for flexibility; test data uses the four main currencies.

**Learned**
- A column's technical name is fixed at creation. Renaming the display label isn't enough if the name is wrong, so I deleted and recreated a mislabelled column early, before anything depended on it.
- Lookup columns can default to optional; required relationships need setting explicitly.
- Choice values (not labels) are what flows compare, so shared lists must be created in the same order and checked.
- The grid editor saves each cell automatically; multi-line text is easier to edit through the row form.

---

## 13/10/2026 — Test data, rules and knowledge document

**Done**
- Added a **Run date override** to Business Setting, set to 03/11/2026, so every test run treats the same day as "today" and results are repeatable.
- Wrote the deterministic chasing rules ([docs/chasing-rules.md](docs/chasing-rules.md)) and generated 12 clients and 30 invoices designed around them, with the expected outcome for each invoice in [eval/test-set.csv](eval/test-set.csv): 17 drafts, 7 escalations, 6 not listed.
- Checked the expected outcomes with an independent script that re-applies the rules: 0 mismatches.
- Wrote the one-page payment terms document the agent will use for late-fee wording.
- Imported clients and invoices into Dataverse and spot-checked dates against the source.

**Problems and fixes**
- **Lookup import failed:** the Excel importer expects the client's internal ID, not its name ("not a valid primary id Guid value"). Fixed by exporting clients with their IDs and building the invoice file with IDs in the Client column.
- **Dates shifted by one day:** the importer read dates as local midnight and converted to UTC, so dates during Irish Summer Time landed a day early. Fixed by importing dates at midday. Caught by checking imported values against the source, not just row counts.
- **Contradiction in the user stories** (queried invoices both "not listed" and "escalated") resolved: queried invoices are listed only as escalations.

**Learned**
- Define expected outcomes before building the agent; it turns evaluation into a comparison rather than a judgement call.
- Always verify imported data field by field. A successful import can still be wrong, and a one-day date shift would silently break stage logic.
- The import error log only opens after signing in to the environment's own URL.

---

## 14/10/2026 — GetDueInvoices flow (rule engine)

**Done**
- Built the `GetDueInvoices` flow in Power Automate: read settings, work out the run date, read invoices, then `Select` → `Select` → `Filter array` → `Select` to add facts, stage, filtering and action/reason. No loops.
- Before building, simulated every expression against the test data (30/30), so problems in Power Automate could only be clicking or pasting issues.
- Tested in stages: after the Base step (30 invoices × 9 fields checked against the source: 0 mismatches), then the full flow.
- **Result: 30/30 test invoices match the expected outcome** (24 listed: 19 Draft, 5 Escalate; 6 not listed). Scored in [eval/results-getdueinvoices.csv](eval/results-getdueinvoices.csv).

**Problems and fixes**
- **Run date stored as 11 March, not 3 November:** typing 03/11/2026 in a US-format form saved it as month/day. Caught by reading the flow's raw output. Fixed by using the date picker.
- **Expressions pasted as plain text:** in the new designer, expressions must go through the `fx` editor (or advanced mode with a leading `@`); pasted straight into a box they become literal text.
- **Lost connection on a new Dataverse action:** fixed with "Change connection reference".
- The classic designer's search didn't show the Dataverse actions; switching to the new designer (adding `?v3=true` to the flow's edit URL) fixed it.

**Learned**
- Test after each tricky step, not only at the end, so a failure points to one step.
- Reading a flow's raw output is the most reliable way to see what the data really is.
- `Select` and `Filter array` can replace most loops: faster, and the whole logic stays visible.

**Known issue**
- Reason text says "1 days overdue"; grammar fix planned when the flow is connected to the agent.

---

## 14/10/2026 — Remaining agent tools

**Done**
- `GetClientContext`: client profile, late-fee flag, notes (with an "untrusted data" warning built into the output) and invoice history. Tested on Northgate (injection note).
- `SaveDraft`: checks each draft against the invoice record and business rules before saving. Tested both paths: a correct draft is saved as Pending; the same draft with a wrong amount is rejected and nothing is saved.
- `EscalateToOwner`: records an escalation with its reason, no draft. Tested on KL-2026-0115 (final notice already sent).
- Each flow's expressions were simulated before building ([flows/](flows/)).

**Problems and fixes**
- **NotFound on a client lookup:** the pasted ID had leading spaces. Fixed by trimming every ID input, which also protects against an agent adding spaces.
- **Invalid filter expression:** mixing typed text and an expression chip in one field failed; one `concat()` expression worked.
- **Name mismatch** (`GetInvoices` vs `GetInvoice`) broke references; renaming the action fixed it. Action names are part of the logic.
- **Test panel reran old inputs**, so a "negative" test passed with the old, correct draft. Caught by checking the run's actual inputs. Negative tests now use fresh manual inputs.

**Learned**
- A guardrail is only proven when its failure path is tested, not just its success path.
- Check what a test actually received before trusting its result.

---

## 14/10/2026 — Agent, evaluation and review app

**Done**
- Created the agent in Copilot Studio with instructions v1.0, then v1.1 ([prompts/](prompts/)).
- Swapped the four flows to the agent trigger with "Respond to the agent", converted them to Copilot Studio agent flows, and added them as tools. Added the payment terms document as knowledge; removed web search.
- Started an admin Copilot Studio trial (25,000 Copilot Credits) and allocated it to the developer environment, with pay-as-you-go left off.
- **Run 1 (v1.0):** 87.5% correct tool path; two invoices confused, one silently missed while the summary said "saved". **Run 2 (v1.1):** 95.8%; every invoice has a record, no invented content. Facts, the late-fee rule and injection resistance held at 100% in both runs ([run 1](eval/runs/run1-report.md), [run 2](eval/runs/run2-report.md)).
- Built the **Reminder Review** app from a written description with Copilot Studio's App (preview): a pending queue with escalations flagged, a detail screen with draft and editable copy, Approve / Approve with edits / Skip, and an "is it still unpaid?" confirmation. Loads real Reminder Log data; approval saves.

**Found**
- The reviewer approved KL-2026-0126 (the hardship client the agent should have escalated) because the review screen doesn't show client notes. Human review only works if the reviewer sees the context.

**Open items**
- Show client notes and a "sensitive client" warning on the review screen.
- Add a rule-based "Handle personally" flag on Client so sensitive clients are always escalated, rather than relying on the LLM.
- EscalateToOwner doesn't set Outcome = Escalated.
- Verify the Skip and Approve-with-edits paths save correctly.
- Send-and-log flow (approved, then Sent); export the solution to `solution/`; screen recording; LinkedIn post.

---

## 17/10/2026 — Solution export and review app redesign

**Done**
- Exported the unmanaged solution to [`solution/`](solution/): four tables, four agent flows, the agent with its tools and knowledge. Checked for credentials before committing. The generated review app doesn't export with the solution.
- Checked credit usage: the admin centre still reports 0 of 25,000 Copilot Credits consumed after two evaluation runs (recorded as "not yet reported", not "free").
- Redesigned the Reminder Review app around the reviewer ([app](apps/reminder-review.md)): removed the process banner, internal reference, stage, prompt version and "Validation passed"; split the queue into **Ready to send** and **Handle personally**; one editable text box with the original draft kept in the data; one Approve button that records Approved or Edited.
- Added formula columns to the Reminder Log (client name, amount, due date, contact name and email, client notes) so the app can show client context without new data connections ([data model](docs/data-model.md)).

**Problems and fixes**
- **App builder couldn't connect new tables:** "live-data connectivity isn't enabled in this session", even after being given the exact schema. Workaround: bring the data into the table the app already uses, using Dataverse formula columns.
- **Formula failed on the first try:** some column display names have a trailing space (`'Client '`, `'Amount '`). Fixed by building formulas from the editor's suggestion list instead of typing.
- **A new column silently switched to Decimal** when named "Amount". Caught before saving; each column's data type now gets checked before Save.
- **Currency wouldn't come through:** formula columns can't read choice columns, either via a lookup or with `Text()` on the same table. Workaround: derive currency from the client's country, which holds for the demo data (documented as a limitation).

**Learned**
- A generated app is a first draft built around the data model. It needed a design pass to be usable: hide the mechanics, separate different jobs, show what the decision needs.
- When a tool blocks one route (connecting tables), move the data to where the tool can reach it.
- In production, approvals belong where the user already works (Teams or Outlook), not in a separate app.

**Open items**
- Confirm the Currency column saves and the formula columns fill in; then show them in the app, with clickable "Handle personally" rows and a "What did you do?" note.
- Test edited approval and Skip on version 2.
- Rule-based "Handle personally" flag on Client; EscalateToOwner to set Outcome = Escalated.
- Send-and-log flow; screen recording; LinkedIn post.
