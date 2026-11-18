# Lookups: names → IDs

Never ask the user for UUIDs. Ask for names, look the IDs up, show names only. One match → confirm it; several → numbered pick list.

## Customer → `customerId`

Ask: "Which customer is this for? Type the full name, or at least the first three letters."

`customers___list_customers` with `query`: the text as typed, `pageNo: 1`, `pageSize: 10`. Each word matches as a prefix; all words must match; results are ranked. Show `name` only.

- One match: confirm ("Oak Plaza HOA, correct?").
- Several: pick list. More than 10 in `meta`: ask for more of the name.
- None: ask for another spelling. If it really does not exist, offer to create it (read `references/customer.md` and follow it), then continue with the new customer selected.

## Property → `propertyId`

`properties___list_properties` with `customerId`, `orderBy: {column: "name", direction: "ASC"}`, `pageSize: 50`. Only active properties come back. Show "Name (street, city)".

- One: confirm. Several: pick list (if the user already named one, pass it as `query`).
- More than 50: ask for a name and pass it as `query`.
- None: the customer has no active properties; stop. A property must be added in the web app first.

## Contacts → `primaryContactId`

`contacts___list_contacts` exactly like the web app: `accountId` (the customer ID), `pageNo: 1`, `pageSize: 1000`, nothing else.

The result includes inactive contacts: offer only `isActive: true`, sorted by first name, shown as "First Last (email)". Only inactive contacts = no contacts. More than 15: ask for a name and match within the result.

No suitable contact: offer to create one (read the contact flow (`references/contact.md`), customer pre-filled, do not ask for it again), then use the new contact.

## Employees → supervisor, administrator, sales rep, technician

`employees___list_employees` **once** per conversation: no `query`, `orderBy: {column: "firstName", direction: "ASC"}`, `pageSize: 500`. Keep and reuse the result. Each employee has `id`, `firstName`, `lastName`, `email`, `role` (CXO, EXECUTIVE, ADMINISTRATOR, SUPERVISOR, TECHNICIAN) and `status`. The result also contains employees with `status: "SIGNUP PENDING"`: offer **only `status: "ACTIVE"`** (the server rejects anyone else for a job role).

- Offer the employees whose role matches the field first ("First Last (email)", numbered). The user may type a name instead: case-insensitive match on first name, last name or both, partial allowed ("raj" → "Raj Kumar").
- If the pick's role differs from the field, mention the role and ask to confirm; do not block.
- More than 15 for a role: ask for a name.
- Call the tool again only when `meta` shows the list was cut off and a typed name is missing (`query: <name>`, `pageSize: 10`), or the first call failed.

## Jobs → `jobId`

`jobs___list_jobs` with `customerId` and/or `propertyId` (from the lookups above) or `query`: words from the job name as typed; `pageNo: 1`, `pageSize: 10`. Show "Job name — frequency — service type — status". One match: confirm. Several: pick list. None: ask for the customer or other words. Details of the chosen job: `job-details___get_job` (it also returns the job's `customer`, `property`, contacts and visit window, which the work-order flow needs).

## Invoices → `invoiceId`

`invoices___list_invoices` with `customerId` or `jobId` (from the lookups above) and/or `query`: the invoice number or name as typed; `pageSize: 10`, newest first. Show "Number — date — total — payment status". One match: confirm. Details: `invoice___get_invoice`. Not available to TECHNICIAN / SUPERVISOR users.
