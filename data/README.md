# Data

Synthetic data for Kestrel Lane Print & Signage. All names, emails and amounts are fictional; emails use the reserved `.example` domain.

| File | Rows | Import into |
|---|---|---|
| `clients.csv` | 12 clients (AU, NZ, UK, US) | Client table |
| `invoices.csv` | 30 invoices | Invoice table (import after clients; the Client column matches client names) |

Dates are ISO format (YYYY-MM-DD) and built around the run date **03/11/2026** (Business Setting → Run date override). Expected results per invoice are in [`eval/test-set.csv`](../eval/test-set.csv); the rules behind them are in [`docs/chasing-rules.md`](../docs/chasing-rules.md).

**Coverage:** every reminder stage, window boundaries, paid, queried, promised (future and missed), part-paid, zero balance still open, already contacted today, already reminded at the current stage, stage skipping, late fee allowed vs excluded, a hardship note, and a prompt-injection note.
