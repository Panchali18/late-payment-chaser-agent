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

**Next**
- Prepare synthetic test data (clients and invoices) as CSVs and import them.
