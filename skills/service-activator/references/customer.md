# Customers

A customer is the account that is invoiced; it owns properties, contacts and jobs. The lookup rules, error handling and record links are in the sections at the end of this guide.

## When to use this flow

The user wants to see, find, list, create or update a **customer**: "find Oak Plaza", "show me Oak Plaza HOA's details", "list my customers", "create customer …", "change the net term / address / billing contact of Oak Plaza HOA".

## Tools

| Tool | Use |
|---|---|
| `customers___list_customers` | find customers by text; one page with name, phone, email, address, status |
| `customer-details___get_customer` | one customer with every field and its main, billing and billing CC contacts |
| `customer___create_customer` | create (Create a customer below) |
| `customer___update_customer` | change fields (Update a customer below) |
| `contacts___list_contacts` | the customer's contacts, for the contact fields of an update |

## Find a customer

1. `customers___list_customers` with `query`: the text as typed (full name or at least the first three letters), `pageNo: 1`, `pageSize: 10`. "List my customers": no `query`, `pageSize: 10`, then offer the next page if `meta` shows more.
2. Show a numbered list of names (add the city when several look alike). One match: confirm it.
3. Details: `customer-details___get_customer` with the `id`, then show it **always in the layout below** (it mirrors the web app's customer page). Never show IDs or `idHash`.

### Customer details layout

Use exactly these sections, in this order, with these labels. Show every row even when empty (`—`); never add, drop or rename rows.

```
## <name>
<street>, <city>, <state>, <country>, <zipCode>
Status: Active | Created On: MM/DD/YYYY | Last Updated On: MM/DD/YYYY

**Details**
| Time zone | <friendly name>     |
| Website   | <website or —>      |
| Phone     | <phone or —>        |
| Mobile    | <mobile or —>       |
| Email     | <email or —>        |

**Other Info**
| Send Overdue Invoice Payment Reminders | Yes / No        |
| Net Term                               | <netTerm>       |
| CPI Increase Percent                   | <n>%            |

**Notes**: <otherInformation or —>

Check it here: https://app.myserviceactivator.com/customers/<id>
```

Formatting rules:
- Address line: only the parts that are present, comma separated; no address → `No office address`.
- Status: `isActive` true → Active, false → Inactive.
- Dates: `createdAt` / `updatedAt` as `MM/DD/YYYY` (like the web app).
- Time zone: friendly name with abbreviation, e.g. `America/Los_Angeles` → `Pacific Standard Time (PST)`, `America/Denver` → `Mountain Standard Time (MST)`, `America/Chicago` → `Central Standard Time (CST)`, `America/New_York` → `Eastern Standard Time (EST)`; anything else as given.
- Yes / No for booleans; empty strings and `null` → `—`.
- After the block, offer in one line: "Want to see this customer's contacts, properties, jobs or invoices?" (use the matching flow).

For the customer's properties, jobs, contacts or invoices use the matching flow.

## Create a customer

Goal: collect everything the web app's customer form collects in **one message**, fill the defaults, confirm, then call `customer___create_customer` once.

### Step 1: ask for everything at once

If the user already gave the customer name (e.g. "create customer Oak Plaza HOA"), keep it and still show the form for the rest.

Send this as **one** message (fill in anything the user already said):

> Please give me the customer details. Only the name is required; leave out or say "skip" for anything you don't need.
>
> **Customers info**
> - Customer account name *(required)*:
> - Website:
>
> **Office address** *(optional)*
> - Street:
> - City:
> - State:
> - Zip code:
>
> **Other info**
> - Net term (DUE ON RECEIPT, 15 DAYS, 30 DAYS, 45 DAYS, 60 DAYS, 90 DAYS) — default **30 DAYS**:
> - CPI increase percent — default **0**:
> - Send overdue invoice payment reminders (yes/no) — default **yes**:
> - Notes (up to 500 characters):

The user may answer in any order or as free text ("Oak Plaza HOA, oakplaza.com, 12 Main St Austin Texas 78701, net 45"). Read what you can, then ask **once** for anything required that is missing or invalid. Do not ask about fields the user skipped.

### Step 2: map the answers

| Form field | Tool field | Rules |
|---|---|---|
| Customer account name | `name` | Required, not empty. |
| Website | `website` | Plain string, as typed. |
| Street / City / State / Zip code | `officeAddress.street` / `.city` / `.state` / `.zipCode` | Send `officeAddress` only if at least one address field was given. |
| (not on the form) | `officeAddress.country` | Always `"US"` when `officeAddress` is sent. |
| State | `officeAddress.state` | Full state name, as the web app sends it: "TX" / "tx" / "texas" → `"Texas"`. |
| Zip code | `officeAddress.zipCode` | `12345` or `12345-6789`; otherwise ask again. |
| Net term | `netTerm` | One of `DUE ON RECEIPT`, `15 DAYS`, `30 DAYS`, `45 DAYS`, `60 DAYS`, `90 DAYS`. "net 45", "45" → `45 DAYS`; "on receipt" → `DUE ON RECEIPT`. Default `30 DAYS`. |
| CPI increase percent | `cpiIncreasePercent` | Number ≥ 0 with at most one decimal ("3.5%" → `3.5`). Default `0`. |
| Send overdue invoice payment reminders | `isAutoInvoiceOverdueSummaryEmail` | yes → `true`, no → `false`. Default `true`. |
| Notes | `otherInformation` | Plain string, at most 500 characters; if longer, ask the user to shorten it. |

Leave out any field the user skipped (other than the defaults above). Never invent values.

### Step 3: check for an existing customer

Call `customers___list_customers` with `query`: the customer name, `pageNo: 1`, `pageSize: 10`.

- A customer with the same name (ignoring case) exists: show it and ask "A customer named X already exists. Create another one anyway?" Continue only on yes.
- Otherwise continue. If the call fails, say so in one line and continue.

### Step 4: confirm

Show a short summary in plain words (only the fields that will be sent, with defaults marked "(default)"), then ask: "Create this customer?" Call the tool only after a clear yes.

### Step 5: create

Call `customer___create_customer` once with the mapped fields. Do not send `customerId`.

- **200** (returns `id`): reply in one line: "Created customer **<name>**.", then the customer link (see `references/links.md`). Keep the `id`; if this was started from the job flow (`references/job.md`), return to it with this customer selected.
- **400:** explain `errorDescription` in plain words (e.g. "Invalid zip code" → "the zip code must be 5 digits, or 5+4 like 12345-6789"), ask for the fix, and retry only after the user answers.
- **401:** the session is not authorized; ask the user to reconnect or sign in again, then stop.
- **500:** do not retry automatically (the customer may have been created). Run the Step 3 lookup again; if the customer now exists, report it. Otherwise ask the user whether to try again.

Do not show raw error bodies unless the user asks.

## Update a customer

1. Find the customer (above) and show the current values.
2. Ask what to change, or apply what the user already said. Field rules are the **same as in Create a customer** (Step 2 mapping). Extra fields only an update can set:

| Field | Rule |
|---|---|
| `mainContactId`, `billingContactId`, `billingCcContactId` | Must be contacts of **this** customer: `contacts___list_contacts` with `accountId`, pick by name. |
| `isActive` | true / false; "deactivate" = `false`. Ask to confirm: deactivating hides the customer from lookups. |
| `billingDay` (`1st`, `15th`), `billingMode` (`E-MAIL`, `POST`), `billingCategory` (`SEPARATE INVOICE FOR EACH JOB`, `ONE INVOICE FOR ALL JOBS`), `parentCustomerId`, `billToName`, `timeZone` | exact values from the tool description; `parentCustomerId` from a customer lookup. |
| `name` | cannot be empty. |

3. Payload: `customerId` plus **only** the changed fields; `officeAddress` only when an address part changes (then send the full address object with `country: "US"`).
4. Show only the changed lines, ask for **yes**, call `customer___update_customer` once. On 200: "Updated **<name>**." plus the customer link.
