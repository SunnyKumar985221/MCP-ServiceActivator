# Properties

A property is a service location of a customer. The lookup rules, error handling and record links are in the sections at the end of this guide.

## When to use this flow

The user wants to see or find a **property**: "which properties does Oak Plaza HOA have", "show me the Oak Plaza North address", "what's scheduled at Oak Plaza North on Friday", "is Hall Season Four active".

Properties are **created and edited in the web app only**; there is no create or update tool. If the user asks to add or change one, say so and give the properties link (`references/links.md`).

## Tools

| Tool | Use |
|---|---|
| `properties___list_properties` | the properties of a customer (active only by default), or a search by name |
| `property-details___get_property` | one property: name, address, coordinates / geofence, access notes, status, customer; with `workOrderDate`, the visits at it on that date |

## Find a property

1. Ask for the customer (`references/lookups.md` → Customer) unless the user named the property directly; then `properties___list_properties` with `customerId` (or `query` with the typed name when there is no customer). Pass `allowInactive: true` only if the user asks about inactive / old properties.
2. Show "Name (street, city)" as a numbered list; one match → confirm it.
3. Details: `property-details___get_property` with `propertyId`. Show name, full address, status, customer and access notes in plain words. Say "no coordinates / geofence" when the geometries are empty; never print raw coordinates unless asked.

### Work orders at a property

For "what's scheduled at <property> on <date>": the same call with `workOrderDate` (`YYYYMMDD`). Show one line per work order: job name, visit window, technicians, status. Nothing scheduled: say so. For more about one visit, use the work-order flow (`references/work-order.md`).

## Rules

- Never ask for or show the property ID; show names and addresses.
- `propertyId` always comes from `properties___list_properties`.
- Dates in plain language ("next Friday") → `YYYYMMDD`; confirm the resolved date in the reply.
- A property is found only under its own customer: if the user names the wrong customer, say which customer it belongs to (from the list result).

## Errors specific to properties

| Error | Meaning / fix |
|---|---|
| Invalid work order date | `workOrderDate` must be `YYYYMMDD`. |
| No property found | Not visible to this user, or inactive — retry with `allowInactive: true`. |
