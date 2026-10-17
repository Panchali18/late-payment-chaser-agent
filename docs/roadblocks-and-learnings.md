# Roadblocks and Learnings

A running record of what got in the way while building this project, how it was resolved, and what it teaches. Written for future builders, and as raw material for LinkedIn posts.

---

## Platform and licensing

### 1. A personal Microsoft account can't build on Power Platform
**What happened:** Power Apps Developer Plan and Copilot Studio need a work or school account.
**Fix:** Created a free Azure account, which creates a Microsoft Entra tenant, then created a separate "builder" user in it.
**Learning:** The real entry cost of "free" enterprise AI tooling is identity setup, not money. Budget an hour for it.

### 2. The Copilot Studio editor opened, but couldn't save an agent
**What happened:** "You don't have permission to create agents. User license not found."
**Fix:** Signed up for the Copilot Studio trial through Microsoft's sign-up link with the builder account.
**Learning:** Being able to open a tool is not the same as being licensed to use it.

### 3. New Copilot Studio can't use Power Automate flows as tools
**What happened:** Four tested Power Automate flows didn't appear in the agent's tool list: "Only workflows that use the 'When an agent calls the workflow' trigger are shown. Power Automate cloud flows are not supported."
**Fix:** Converted each flow to a Copilot Studio agent flow (flow details → Plan → Copilot Studio). A one-way change; the flow must be in a solution first.
**Learning:** Platform boundaries move. Build logic so it can be lifted (expressions documented, tested independently), and the move is a setting change, not a rebuild.

### 4. Zero credits: the trial unlocked the editor but not the ability to test
**What happened:** First test message: "You need credits to continue… This environment is out of credits" (EnforcementUsageCredits). The Power Platform admin centre showed 0 Copilot Credits: purchased, assigned and pay-as-you-go.
**Context:** From 1 September 2026, agents built with Copilot Studio's new GitHub Copilot harness consume Copilot Credits for building, previewing and testing, including in developer and trial environments.
**Fix (in progress):** Admin-started "Microsoft Copilot Studio Trial" in the Microsoft 365 admin centre Marketplace (EUR 0.00, 1 month), then allocate capacity to the environment.
**Learning:** For an AI product owner, "free trial" needs unpacking: editor access, user licence, tenant capacity and environment allocation are four separate things. Cost starts at testing, not at go-live, which changes how you plan evaluation.

---

## Data

### 5. Lookup import failed
**What happened:** Excel import of invoices failed: "not a valid primary id Guid value."
**Fix:** Exported clients with their IDs and imported invoices referencing IDs, not names.
**Learning:** Relational imports need keys, not labels.

### 6. Dates shifted by one day on import
**What happened:** Dates imported as local midnight were converted to UTC and landed a day early.
**Fix:** Imported dates at midday. Caught by checking values field by field, not row counts.
**Learning:** A successful import can still be wrong. Verify data, not status messages.

### 7. Run date saved as 11 March instead of 3 November
**What happened:** Typing 03/11/2026 in a US-format form stored month/day.
**Fix:** Used the date picker; confirmed by reading the flow's raw output.
**Learning:** Check the stored value, not what you think you typed.

---

## Building flows

### 8. Expressions pasted as plain text
**Fix:** Expressions go through the fx editor (or advanced mode with a leading @).

### 9. Classic designer search didn't show Dataverse actions
**Fix:** Switched to the new designer by adding `?v3=true` to the flow's edit URL.

### 10. Small slips with big effects
- A trailing "s" (GetInvoices vs GetInvoice) broke every reference to an action.
- Leading spaces in a pasted ID caused "NotFound"; now every ID input is trimmed.
- The test panel silently reran the previous inputs, so a negative test "passed" with the old, correct data.

**Learning:** Test the failure path, and check what a test actually received before trusting its result.

---

## Approach that paid off

- **Rules before AI:** the deterministic rule engine passed 30/30 test invoices before any LLM was involved.
- **Simulate before building:** every Power Automate expression was run against the test data in a small simulator first, so in-tool problems were clicking problems, not logic problems.
- **Guardrail around the LLM:** SaveDraft rejects any draft whose invoice number, amount, currency or due date doesn't match the record, or that mentions a late fee where it isn't allowed. Tested on both the pass and fail paths.

---

## Post ideas

1. **"Free" AI trials have four layers:** editor access, user licence, tenant capacity, environment allocation. Testing costs money from day one.
2. **Rules first, AI last:** why the LLM in my agent only writes words, and what that did for testability.
3. **My agent can't save a wrong number:** a deterministic guardrail around an LLM, and the test that proved it.
4. **Three bugs that weren't bugs:** a date stored as March, an ID with spaces, a test that reused old inputs. Verify what the system received.
5. **When the platform moves under you:** Copilot Studio stopped accepting Power Automate flows mid-build, and why documented, tested logic made it a 10-minute fix.

---

## Agent evaluation (run 1)

### 11. The chat summary didn't match what the agent did
**What happened:** The agent's summary reported "Draft saved" for KL-2026-0130, but no draft exists. It also labelled KL-2026-0123 as a different client, and skipped KL-2026-0125 entirely.
**How it was caught:** Scoring against the Reminder Log export, not the chat.
**Learning:** Evaluate an agent on its actions, not its account of its actions. The guardrails (fact validation, no send tool) meant none of these errors could reach a client, but a silent miss is still a business failure.

### 12. Context bleed between clients
**What happened:** An escalation reason for one client quoted another client's notes. Two similar invoices (same stage, same days overdue) were swapped.
**Learning:** Long multi-item runs are where LLM agents slip. Process one item at a time, restate identifiers before each tool call, and add a deterministic reconciliation check after the run.

### 13. Credits
**What happened:** 25,000 trial Copilot Credits came from an admin-started trial in the Microsoft 365 admin centre, then had to be allocated to the environment manually in the Power Platform admin centre.

## More post ideas

6. **My agent said it saved 17 drafts. It saved 15.** Why you score agents on the database, not the chat.
7. **87.5% isn't a pass:** what one evaluation run taught me about where rules end and the LLM begins.

### 14. Human review without context isn't a safeguard
**What happened:** In testing the review app, the reviewer approved a friendly reminder to the hardship client, the same case the agent had missed, because the review screen showed the draft and invoice but not the client's notes.
**Learning:** Human-in-the-loop only catches what the human can see. Design the review screen around the decisions the reviewer has to make, not just the AI's output.

---

## Review app and data (17/10)

### 15. The generated app showed the system, not the job
**What happened:** The app generated from a written brief worked, but showed a process diagram, an internal reference ("KL-2026-0123 | Gentle | 14/10/2026"), stage names, a prompt version and a "Validation passed" column that was always Yes for drafts and No for escalations (which looked like an error). Drafts and escalations shared one table, so half the rows had a blank reason.
**Fix:** A design pass. Two tabs (Ready to send / Handle personally), internals hidden, one editable text box, one Approve button.
**Learning:** AI app generators build from the data model. The product owner's job is to rebuild the screen around the decision the user has to make.

### 16. The app builder couldn't add tables
**What happened:** Copilot Studio's App (preview) refused to add the Invoice and Client tables ("live-data connectivity isn't enabled in this session"), even when given the exact schema.
**Fix:** Added formula columns to the Reminder Log that pull client and invoice details through the existing lookup, so the app reads everything from one table.
**Learning:** When a tool blocks a route, move the data to where the tool can reach it.

### 17. Hidden spaces and unsupported types in formula columns
**What happened:** `'Invoice'.'Client'.'Client name'` failed because the real display name is `'Client '`, with a trailing space. A column named "Amount" silently switched itself to Decimal. Currency, a choice column, couldn't be read at all.
**Fix:** Built formulas by picking from the suggestion list, checked each data type before saving, and derived currency from the client's country.
**Learning:** Small schema decisions (a trailing space, choice vs text) limit what you can do later. Store codes like currency as plain text when other features will need to read them.

### 18. Where should approval live?
**Insight:** A small-business owner won't open a separate app every morning. In production, approvals belong in Teams or Outlook, with the app as the full view behind them.

### 19. A screen figure that contradicted a correct AI draft
**What happened:** For a part-paid invoice, the review screen showed the invoice total (NZD 3,600) while the draft correctly asked for the outstanding NZD 2,400. A reviewer would have assumed the AI got it wrong, or "corrected" it to the wrong figure.
**Fix:** Show Outstanding (Amount − Amount paid).
**Learning:** The review screen has to use the same definitions as the AI. Otherwise human review adds errors instead of catching them.

### 20. Saved isn't published
**What happened:** A fix to EscalateToOwner had been saved as a draft in an earlier session but never published, so the agent kept running the old version. Opening the flow from Power Apps showed the classic designer, which can't edit agent-flow drafts.
**Fix:** Opened it in Copilot Studio and published.
**Learning:** After any change, check what is live, not what is saved.

### 21. A judgement moved from the LLM to a rule
**What happened:** In both evaluation runs, the LLM read "handle personally" in a client's notes only some of the time.
**Fix:** A yes/no flag on the client, checked by the rule engine, added as new steps without touching tested expressions. Both invoices now escalate every time.
**Learning:** When an LLM decision has to be right every time, and a rule can make it, use the rule.

## Post ideas (continued)

8. **The AI built my app in two minutes. Making it usable took an hour:** what I removed from a generated review screen, and why.
9. **Human-in-the-loop is a design problem:** my reviewer approved the one reminder the agent got wrong, because the screen didn't show the client notes.
10. **Approvals belong where people already work:** why the real version of my review app is a Teams message.
11. **My agent was right; my screen made it look wrong:** the part-paid invoice that would have turned human review into a source of errors.
12. **From prompt to rule:** the one decision I took away from the LLM, and why.
