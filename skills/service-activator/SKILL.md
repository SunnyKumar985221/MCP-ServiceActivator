---
name: service-activator
description: Work with Service Activator (field service management) through its MCP tools - find, view, create and update service jobs (one-time or recurring), work orders (visits), customers and contacts; look up properties and invoices. Use when the user asks about Service Activator jobs, work orders, visits, customers, contacts, properties, invoices or outstanding amounts. Asks in plain language, looks IDs up by name, validates like the web app and confirms before saving.
---

# Service Activator

Goal: the user never needs to know field names or IDs. Ask in plain language, look IDs up yourself, validate, confirm, then save.

## Route the request

Read the reference file for the matching row (plus the shared files it points to), then follow it:

| User wants to… | Read |
|---|---|
| see, find, list, create or update a **job** (one-time or recurring service contract) | `references/job.md` |
| see, find, create, reschedule or update a **work order** (one visit of a job) | `references/work-order.md` |
| see, find, list, create or update a **customer** (customer account) | `references/customer.md` |
| see, find, list, create or update a **contact** (person at a customer) | `references/contact.md` |
| see, find or list a **property** (service location), or what is scheduled there on a date | `references/property.md` |
| see, find or list **invoices**, outstanding amounts | `references/invoice.md` |

Shared rules used by every flow:

- `references/lookups.md`: turning names into IDs (customer, property, contacts, employees, jobs, invoices)
- `references/errors.md`: what to show and do when a tool returns an error
- `references/links.md`: the web app link to give after a create or update

Unclear request: ask which of these the user means. Not covered by any flow (deleting or cancelling records, payments, timesheets, crews, employees' details, partners, quotes): say the tools cannot do it and point to the web app (`references/links.md`).

Handoffs: a flow may need another one (job → new customer or contact; work order → job). Read that file, finish it, then return to the original flow with the new record selected.

## Tools by entity

The tools come from the Service Activator MCP server (`https://mcp.abren.ai/mcp`) and are named `<target>___<tool>`; your client may add a prefix (e.g. `mcp__service-activator__job___create_job`). Match on the part shown.

| Entity | Read / list | Create / update |
|---|---|---|
| Job | `jobs___list_jobs`, `job-details___get_job` | `job___create_job`, `job___update_job` |
| Work order | `work-order-details___get_work_order` (find via `property-details___get_property` + `workOrderDate`, or `job-details___get_job` → `todayWorkOrder`) | `work-order___create_work_order`, `work-order___update_work_order` |
| Customer | `customers___list_customers`, `customer-details___get_customer` | `customer___create_customer`, `customer___update_customer` |
| Contact | `contacts___list_contacts`, `contact-details___get_contact` | `contact___create_contact`, `contact___update_contact` |
| Property | `properties___list_properties`, `property-details___get_property` | web app only |
| Invoice | `invoices___list_invoices`, `invoice___get_invoice` | web app / batch only |
| Employees | `employees___list_employees` (lookup for supervisors, administrators, technicians) | — |

## Rules for every flow

- Never ask for or show UUIDs. Ask for names, look the IDs up, show names. Several matches: numbered pick list.
- Accept plain language ("next Tuesday", "$1,420", "3 hours", "8 to 5") and convert it yourself; confirm the resolved value in your reply.
- Reads: show the fields a person cares about in plain words (names, dates, amounts, status); lists default to 10 rows; offer the next page only when the tool's `meta` says there is one.
- Offer "skip" for optional fields; never invent values for skipped fields.
- Validate before calling a create or update tool, using the flow's rules.
- Updates send the record ID plus only the fields that change; show the changed lines and create or update only after a clear yes.
- Call each create or update tool once per request; never retry silently.
- After a successful create or update, end with the record link (`references/links.md`).
- A 401 means the signed-in user's role may not do this (e.g. technicians cannot create jobs, technicians and supervisors cannot list invoices); say so and stop.
