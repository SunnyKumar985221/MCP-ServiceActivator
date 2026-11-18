# Invoices

The lookup rules, error handling and record links are in the sections at the end of this guide.

## When to use this flow

The user wants to see or find **invoices**: "unpaid invoices of Oak Plaza HOA", "invoices for the Oak Plaza window job", "show invoice 10234", "what's overdue", "how much is outstanding for customer X".

Invoices are created by the batch jobs and the web app; there is no create, send or update tool. If asked to create, send, cancel or pay an invoice, say so and give the invoices link (`references/links.md`).

Not available to TECHNICIAN or SUPERVISOR users (the list tool returns 401 for them).

## Tools

| Tool | Use |
|---|---|
| `invoices___list_invoices` | list and filter invoices; returns a page of invoices plus `counts` (by payment status) and `sums` (invoice total, payments, outstanding, average days to pay) for the whole filter |
| `invoice___get_invoice` | one invoice: line items, payments, contacts |

## Find invoices

1. Ask for the scope: a customer (`references/lookups.md` → Customer → `customerId`), a job (→ Jobs → `jobId`), an invoice group, or "all".
2. Call `invoices___list_invoices` with the filters the user implied:

| User says | Filter |
|---|---|
| unpaid / outstanding / due | `paymentStatus: "NOT PAID"` (also `UNDERPAID`, `UNDERPAID/OVERDUE`); for "overdue" use `OVERDUE` |
| paid | `paymentStatus: "PAID"` (or `OVERPAID`) |
| cancelled / processed / created | `status: CANCELED` / `PROCESSED` / `CREATED` |
| recurring / one-time / extra | `type: RECURRING` / `1X ONLY` / `EXTRA ON RECURRING` |
| manual invoices | `isCreatedManually: true` |
| by number or name | `query: "<text>"` |

Default `pageSize: 10`; newest first (`date DESC`) unless the user asks for another order (`orderBy: {column, direction}`, e.g. `dueDate ASC`). Ask for more only when `meta` shows more pages and the user wants them.

3. Show a table: number, date, due date, customer, job, total, status, payment status. Then one line from `sums`: "Outstanding: $12,340.50 across 7 invoices" (use `counts`/`sums`, do not add the rows yourself). Never show IDs.
4. Details: `invoice___get_invoice` with `invoiceId`. Show the line items (description, quantity, amount), payments received, billing contact, and the status in plain words.

## Rules

- `invoiceId`, `customerId`, `jobId` always come from tools, never from the user; an invoice **number** typed by the user goes into `query`.
- Money: show with currency and 2 decimals; dates in the user's words ("12 Oct 2026").
- Status words are exact backend values; the user never sees them raw: say "not paid", "partly paid and overdue", "cancelled".
- Do not compute totals from a page when `sums` exists; `sums` covers the full filter.

## Errors specific to invoices

| Error | Meaning / fix |
|---|---|
| 401 / Unauthorized | TECHNICIAN and SUPERVISOR users cannot list invoices; ask to use another account. |
| Invalid payment status / Invalid status / Invalid type | A filter value is not in the allowed list; re-check the table above. |
| Invalid page no / page size | Integers ≥ 1. |
| No invoice found | Not visible to this user, or the ID came from somewhere other than the list tool. |
