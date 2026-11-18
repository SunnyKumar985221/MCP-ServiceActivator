# Work orders

A work order is one service visit of a job on one date. The lookup rules, error handling and record links are in the sections at the end of this guide.

## When to use this flow

The user wants to see, find, create, reschedule or update a **work order** (a visit): "what's scheduled at Oak Plaza on Friday", "add a visit on the 15th for the Oak Plaza job", "move Tuesday's visit to Thursday", "assign Raj to tomorrow's visit", "check in / check out", "put the visit on hold".

## Tools

| Tool | Use |
|---|---|
| `work-order-details___get_work_order` | one work order by ID: dates, visit window, technicians, crew, supervisor, contacts, notes, billing, status flags |
| `work-order___create_work_order` | add one visit to a job |
| `work-order___update_work_order` | change one visit (reschedule, technicians, check-in/out, hold, notes, billing) |
| `job-details___get_job` | the job a visit belongs to; also returns `todayWorkOrder` |
| `property-details___get_property` with `workOrderDate` | the work orders at a property on one date |

There is **no list tool for work orders**. Find them through the job or the property (below).

## Find a work order

Never ask for a work order ID. Ask for the **job (or customer and property) and the service date**, then:

1. Find the job as in `references/lookups.md` → Jobs. The job gives its `customer`, `property` and `visitWindow`.
2. Get the work orders of that day: `property-details___get_property` with `propertyId` and `workOrderDate` (`YYYYMMDD`). Match by job name when the property has several jobs. For "today's visit" of a job, `job-details___get_job` returns `todayWorkOrder` directly.
3. Several on that date: show a numbered list "Job name — technician(s) — status". One: continue with it.
4. Details: `work-order-details___get_work_order` with the ID from step 2, then show it **always in the layout below**. Never show IDs, `idHash`, `uniqueKey` or `sitemapThumbnailBase64`.

If the date is unknown, ask for it; the tools cannot search by job alone. If nothing is found, say so and give the job link (`references/links.md`) so the user can open the job's visits in the web app.

### Work order details layout

Use exactly these sections, in this order, with these labels. Show every row even when empty (`—`); never add, drop or rename rows. A technician login gets fewer fields: show `—` for the missing ones and skip the **Billing** section.

```
## <name> — <Day DD Mon YYYY>
<jobName> · <customer.name> · <property.name> (<street>, <city>, <state> <zipCode>)
Status: <status> | Created On: MM/DD/YYYY | Last Updated On: MM/DD/YYYY

**Schedule**
| Service Date        | <Day> MM/DD/YYYY                                  |
| Visit Window        | <h:mm AM/PM> to <h:mm AM/PM> [(starts the day before)] |
| Check In / Out      | <h:mm AM/PM MM/DD> / <h:mm AM/PM MM/DD> or —       |
| Estimated Time      | <n> hours <m> minutes                             |
| Access Time         | <averageTimeForAccessMinutes> minutes             |
| Time On Property    | <totalTechTimeSpentMinutes as hours/minutes> or — |
| Rescheduled         | <rescheduledCount> times or —                     |

**Team**
| Technician 1        | First Last [— <n> min] [overtime] [1-hour minimum] |
| Technician 2        | First Last … or —                                  |
| Technician 3        | First Last … or —                                  |
| Technician 4        | First Last … or —                                  |
| Crew                | Assigned / —                                       |
| Supervisor          | First Last or —                                    |
| Partner             | <partner.name> or —                                |

**Contacts**
| Primary Contact     | First Last (email, mobile) or —                   |
| Originated By       | First Last or —                                   |

**Job**
| Frequency           | <jobFrequency>                                    |
| Service Type        | <jobServiceType>                                  |
| Billing Type        | Fixed cost / Time & material                      |
| Cost Per Visit      | $<job.oneServiceCost> or —                        |

**Billing**
| Total To Bill       | $<totalToBillAmount> or —                         |
| Bill Adjustment     | $<billAdjustmentAmount> or —                      |
| Dump Billed / Fees  | $<dumpBilledAmount> / $<dumpFeesAmount> or —      |
| Supplies            | <otherSupplies> — $<otherSupplyCost> (+<supplyMarkUpPercent>%) or — |
| Load                | <loadPercent>% or —                               |
| Billing Information | <billingInformation or —>                         |
| Invoiced            | Yes / No                                          |
| Revenue             | MM/DD/YYYY $<revenue> [billed] [extra work] (one per productSchedules row; none → —) |

**Notes**
| Assignment          | <plain text, max 300 chars, "…" if longer>        |
| Technician Notes    | <technicianNotes or —>                            |
| Access Notes        | <accessNotes or —>                                |
| Sitemap             | Available / —                                     |

Check it here: https://app.myserviceactivator.com/workorders/<id>
```

Formatting rules:
- Status: the `status` field in title case (`PENDING` → Pending, `IN PROGRESS` → In progress, `COMPLETED` → Completed, `INCOMPLETE` → Incomplete). Add "· On hold" when `isOnHold` is true. No `status`: Incomplete if `isIncompleteService`, else On hold if `isOnHold`, else Completed if both check-in and check-out are set, In progress if only check-in, else Pending.
- Dates: `serviceDate` (`YYYYMMDD`) and `createdAt` / `updatedAt` as `MM/DD/YYYY`; add the weekday to the service date.
- Visit window, check-in and check-out: the timestamps (`YYYY-MM-DDTHH:mm:ssZ`) are wall-clock time; show the 12-hour clock **without** time-zone conversion. When the window start falls on the day before `serviceDate`, add "(starts the day before)". `18:00`→`17:59` is the default whole-day window: show "6:00 PM to 5:59 PM (whole day)".
- Technicians: from `technician1Employee`…`technician4Employee`. Add the slot's minutes (`techTimeSpentTech<n>Minutes`), "overtime" (`isOvertimeTech<n>`) and "1-hour minimum" (`isApply1HourMinimumTech<n>`) only when set.
- Crew: the response has only `crewId`, so show "Assigned" (never the ID) or `—`.
- Billing type and cost per visit come from `job` (`isFixedCostBilling`, `oneServiceCost`).
- Money as `$1,420` (2 decimals only when not whole). Minutes ≥ 60 as hours and minutes ("200" → "3 hours 20 minutes").
- Assignment and notes: strip HTML tags.
- If the job name says a different frequency than `jobFrequency`, add a line under the block: "Note: the job name says <x> but the frequency is <y>."
- After the block, offer in one line: "Want to reschedule, change the technicians, or check this visit in/out?" (Update a work order below).

If `get_work_order` fails, say in one line that the visit details can't be loaded right now. If the visit is the job's next or today's visit, show what `job-details___get_job` returns for it (name and date), then give the work order link.

## Create a work order

Call `work-order___create_work_order` once, after the user says yes.

### Step 1: Job (required)

Find the job (`references/lookups.md` → Jobs), then `job-details___get_job`. From it take `jobId`, `customerId`, `propertyId`, the supervisor, the primary contact and the visit window. Show "Adding a visit to **<job name>** (<customer>, <property>)".

### Step 2: Visit details (one message)

Ask these together, with the defaults from the job already filled in:

```
1. Service date? (e.g. 15 Nov 2026)
2. Visit window: <job window, e.g. 6:00 PM to 5:59 PM> — keep or change?
3. Technicians (up to 4) or a crew — names, or "skip" to leave unassigned
4. Anything else: supervisor (<job supervisor>), notes for the technician, access notes, estimated time (<job default>) — or "ok"
```

| Field | Rule |
|---|---|
| `serviceDate` | required, `YYYYMMDD`; a real date. |
| `visitWindowStartTime` / `visitWindowEndTime` | `YYYY-MM-DDTHH:mm:ssZ` on the service date, from the job's window (`18:00:00Z`/`17:59:00Z` by default). Start after end = window starts the day before; say so. Local wall-clock time with `Z` appended, no time-zone conversion. |
| `technician1EmployeeId` … `technician4EmployeeId` | ACTIVE employees (`references/lookups.md` → Employees; offer TECHNICIAN role first). In order, no gaps. |
| `crewId` | only if the user names a crew and its id is known from an existing work order or the job; otherwise ask the user to assign the crew in the web app. |
| `supervisorEmployeeId` | default: the job's supervisor. |
| `primaryContactId` | default: the job's primary contact. |
| `originatedByEmployeeId` | never asked; leave out. |
| `estimatedServiceTimeMinutes`, `averageTimeForAccessMinutes` | whole minutes ≥ 0; defaults from the job. |
| `technicianNotes`, `accessNotes`, `billingInformation`, `assignment` | plain text; leave out when empty. |
| `isOnHold` | only when the user says the visit is on hold. |
| `isClone` / `clonedFromWorkOrderId` | never used by this flow. |

### Step 3: Confirm and create

```
Visit for:      Oak Plaza North 2x Week Window Washing (Oak Plaza HOA)
Date:           Fri 15 Nov 2026 | Window: 8:00 AM to 12:00 PM
Technicians:    Raj Kumar, Amit Shah | Supervisor: Priya Nair
Notes:          gate code 1234
```

"Reply **yes** to create." On 200 (returns `id`): one line "Created the visit on <date> for <job>." plus the work-order link.

## Update a work order

1. Find the work order (above) and show its current date, window, technicians and status.
2. Ask what to change, or apply what the user already said. Send `workOrderId` plus **only** the fields that change.
3. Rules by change:

| Change | Fields | Rules |
|---|---|---|
| Reschedule | `serviceDate`, `visitWindowStartTime`, `visitWindowEndTime` | same formats as create; move the window to the new date; ask whether to keep the same hours. |
| Technicians / crew | `technician1EmployeeId`…`technician4EmployeeId`, `crewId`, `isUpdateTechniciansOnFutureWorkOrders` | ACTIVE employees only. Ask once: "Apply to this visit only, or also to the job's later visits?" → `isUpdateTechniciansOnFutureWorkOrders` true/false (default false). |
| Check in | `checkInTime` | `YYYY-MM-DDTHH:mm:ssZ`; the work order needs a technician or crew first. |
| Check out | `checkOutTime` (+ `checkInTime` if not set) | after check-in and within 24 hours of it. |
| Time spent / overtime / 1-hour minimum | `techTimeSpentTech1Minutes`…`4`, `isOvertimeTech1`…`4`, `isApply1HourMinimumTech1`…`4`, `dailySequenceTech1`…`4` | numbers / booleans per technician slot. |
| Status | `isOnHold`, `isIncompleteService`, `isPhotosUploaded` | booleans; "on hold" is not "cancelled" (there is no delete tool; say so if asked). |
| Billing | `billAdjustmentAmount`, `dumpBilledAmount`, `dumpFeesAmount`, `loadPercent`, `otherSupplies`, `otherSupplyCost`, `supplyMarkUpPercent`, `billingInformation` | numbers ≥ 0, at most 2 decimals. When any amount, `loadPercent` or `supplyMarkUpPercent` is set, `billingInformation` is required (unless the job is 1x Only with fixed cost billing) — ask for it. |
| Notes / contacts | `technicianNotes`, `accessNotes`, `assignment`, `primaryContactId`, `supervisorEmployeeId`, `estimatedServiceTimeMinutes`, `averageTimeForAccessMinutes` | same rules as create. |

4. Show only the changed lines ("Date: Tue 12 Nov → Thu 14 Nov") and ask for **yes**. Call `work-order___update_work_order` once. On 200: "Updated the visit on <date> for <job>." plus the link.

## Validation (before every create or update)

1. IDs come from lookups, never from the user; the property and contact belong to the job's customer.
2. Dates: `serviceDate` strict `YYYYMMDD`; timestamps strict `YYYY-MM-DDTHH:mm:ssZ`.
3. Visit window: start ≠ end; both on (or around) the service date.
4. Employees: ACTIVE only; technician slots filled in order.
5. Check-out needs check-in; check-out ≤ 24 h after check-in.
6. Numbers are JSON numbers ≥ 0 (money: 2 decimals; minutes: whole).
7. Billing amounts need `billingInformation` (see Billing row).
8. Payload: only fields from the tool schema; no empty strings; update sends `workOrderId` + changed fields only.

## Errors specific to work orders

| Error | Meaning / fix |
|---|---|
| 401 / Unauthorized | Technicians see fewer fields and may not create work orders; ask to use another account. |
| Invalid service date / Invalid visit window … | Wrong format; rebuild the value. |
| Invalid check out time | Must be after check-in and within 24 hours. |
| Technician … is not active / is a required field | Pick an ACTIVE employee; check-in needs a technician or crew. |
| Billing information is required | Add `billingInformation` with the amounts. |
| No job found / No property found | Not visible to this user; re-check the job. |
