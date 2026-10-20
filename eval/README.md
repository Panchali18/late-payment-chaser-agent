# Evaluation

`test-set.csv`: 30 synthetic invoices with the expected stage, tool path and outcome for each.
`results-getdueinvoices.csv`: the rule engine's output compared with the test set (30/30 match).
`runs/`: one folder of evidence per agent run: the Reminder Log export, per-invoice scoring and a report. [Run 1](runs/run1-report.md) used prompt v1.0, [run 2](runs/run2-report.md) prompt v1.1.

Agents are scored on the records they wrote, not on their chat summary.
