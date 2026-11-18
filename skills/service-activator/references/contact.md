# Contacts

A contact is a person at a customer (or partner) account. The lookup rules, error handling and record links are in the sections at the end of this guide.

## When to use this flow

The user wants to see, find, list, create or update a **contact**: "who is the contact at Oak Plaza", "show John Smith's phone", "add a contact for Oak Plaza HOA", "change John's email", "mark John as do-not-call", "deactivate a contact".

## Tools

| Tool | Use |
|---|---|
| `contacts___list_contacts` | the contacts of a customer (`accountId`), or a search by text; includes inactive ones |
| `contact-details___get_contact` | one contact with every field and the name of its account (`accountName`) |
| `contact___create_contact` | create (Create a contact below) |
| `contact___update_contact` | change fields (Update a contact below) |
| `customers___list_customers` | the customer lookup |

## Find a contact

1. Usually by customer: find it (`references/lookups.md` → Customer), then `contacts___list_contacts` with `accountId`, `pageNo: 1`, `pageSize: 1000`. By name across customers: `query: "<name>"`, `pageSize: 10`. Use `isActive: true` unless the user asks for inactive contacts.
2. Show "First Last (email) — title" as a numbered list, active ones first. One match: confirm it.
3. Details: `contact-details___get_contact` with the `id`. Show name, title, email, phone / mobile / extension, mailing address, flags (active, do not call, email opt-out, CPI opt-out) and the account it belongs to. Never show IDs.

## Create a contact

Goal: collect everything the web app's contact form collects in **one message**, fill the defaults, confirm, then call `contact___create_contact` once.

### Step 1: ask for everything at once

If the contact is being created from the job flow, the customer is already known: show its name in the form and do not ask for it again.

Send this as **one** message (fill in anything the user already said):

> Please give me the contact details. Required: customer account name, first and last name, and at least one of email, mobile or phone. Leave out or say "skip" for anything you don't need.
>
> **Contact info**
> - Customer account name *(required)*:
> - Name *(required)*: honorific (Mr., Mrs., Ms.) — default **Mr.**, first name, last name
> - Title:
> - Email:
> - Mobile number:
> - Phone number:
> - Ext:
> - Is active (yes/no) — default **yes**:
> - Description:
>
> **Mailing address** *(optional)*
> - Street:
> - City:
> - State:
> - Zip code:
> - Country: **US**
>
> **Additional information**
> - CPI letter opt-out (yes/no) — default **no**:
> - Do not call (yes/no) — default **no**:

The user may answer in any order or as free text ("John Smith, manager, john@oak.com, 555 123 4567, for Oak Plaza HOA"). Read what you can, then ask **once** for anything required that is missing or invalid. Do not ask about fields the user skipped.

### Step 2: find the customer

Unless it came from the job flow, call `customers___list_customers` with `query`: the customer name the user typed, `pageNo: 1`, `pageSize: 10`.

- One match: confirm it ("Oak Plaza HOA, correct?").
- Several: show a numbered pick list of names.
- None: say so and ask for another spelling. If the customer does not exist, offer to create it first (read `references/customer.md` and follow it), then come back.

Keep the customer's `id` as `accountId`. Show names only, never IDs.

### Step 3: map the answers

| Form field | Tool field | Rules |
|---|---|---|
| Customer account name | `accountId` | The customer's `id` from Step 2. Required. |
| (not on the form) | `type` | Always `"CUSTOMER"`. |
| Honorific | `honorific` | `MR.`, `MRS.` or `MS.` ("mr", "Mr" → `MR.`). Default `MR.`. |
| First name / Last name | `firstName` / `lastName` | Both required. If the user gives one full name, split on the last space and confirm. |
| Title | `title` | Plain string. |
| Email | `email` | Must be a valid email address. |
| Mobile number / Phone number | `mobile` / `phone` | `+1` followed by 10 digits. Convert "(555) 123-4567", "555-123-4567", "5551234567" → `"+15551234567"`. Anything else: ask again. |
| Ext | `phoneExtension` | Number. Only with a phone number. |
| Is active | `isActive` | Default `true`. |
| Description | `description` | Plain string. |
| Street / City / State / Zip code | `mailingAddress.street` / `.city` / `.state` / `.zipCode` | Send `mailingAddress` only if at least one address field was given. |
| Country | `mailingAddress.country` | Always `"US"` when `mailingAddress` is sent (fixed in the web app). |
| State | `mailingAddress.state` | Full state name, as the web app sends it: "CA" / "california" → `"California"`. |
| Zip code | `mailingAddress.zipCode` | `12345` or `12345-6789`; otherwise ask again. |
| CPI letter opt-out | `isCpiEmailOptOut` | Default `false`. |
| Do not call | `isDoNotCall` | Default `false`. |

At least one of `email`, `mobile` or `phone` is required; if none is given, ask for one. Leave out any field the user skipped (other than the defaults above). Never invent values.

### Step 4: check for an existing contact

Call `contacts___list_contacts` with `accountId`, `pageNo: 1` and `pageSize: 1000` (nothing else, like the web app).

- A contact with the same first and last name, or the same email, already exists for this customer: show it ("John Smith (john@oak.com) already exists for Oak Plaza HOA") and ask whether to use that one or create another. In the job flow, "use that one" means selecting it as the contact.
- Otherwise continue. If the call fails, say so in one line and continue.

### Step 5: confirm

Show a short summary in plain words (only the fields that will be sent, defaults marked "(default)"), then ask: "Create this contact?" Call the tool only after a clear yes.

### Step 6: create

Call `contact___create_contact` once with the mapped fields. Do not send `contactId`.

- **200** (returns `id`): reply in one line: "Created contact **<First Last>** for <customer>.", then the contact link (see `references/links.md`). Keep the `id`; if this was started from the job flow (`references/job.md`), return to it with this contact selected.
- **400:** explain `errorDescription` in plain words (e.g. "Invalid mobile" → "the mobile number must be +1 followed by 10 digits"), ask for the fix, and retry only after the user answers.
- **401:** the session is not authorized; ask the user to reconnect or sign in again, then stop.
- **500:** do not retry automatically (the contact may have been created). Run the Step 4 check again; if the contact now exists, report it. Otherwise ask the user whether to try again.

Do not show raw error bodies unless the user asks.

## Update a contact

1. Find the contact (above) and show the current values.
2. Ask what to change, or apply what the user already said. Field rules are the **same as in Create a contact** (Step 3 mapping). Extra rules for updates:

| Field | Rule |
|---|---|
| `email`, `phone`, `mobile` | After the change the contact must still have at least one of the three; refuse to clear the last one. |
| `isActive` | "deactivate" = `false`; confirm first, the contact then disappears from pick lists. |
| `type` | `CUSTOMER` or `PARTNER`; never changed unless the user asks explicitly. |
| `isAutomaticBillingSystem` | when `true`, `invoiceEscalationContactId` (another contact of the same account) is required. |
| `ownerId`, `parentEmployeeId` | ACTIVE employees (`references/lookups.md` → Employees). |

3. Payload: `contactId` plus **only** the changed fields; `mailingAddress` only when an address part changes (then the full object with `country: "US"`).
4. Show only the changed lines, ask for **yes**, call `contact___update_contact` once. On 200: "Updated **<First Last>**." plus the contact link.
