# Record links

After every successful create or update, end the reply with a link to the record in the web app:

`Check it here: https://app.myserviceactivator.com/<path>/<id>`

| Record | Link |
|---|---|
| Job | `https://app.myserviceactivator.com/jobs/<jobId>` |
| Customer | `https://app.myserviceactivator.com/customers/<customerId>` |
| Contact | `https://app.myserviceactivator.com/contacts/<contactId>` |
| Work order | `https://app.myserviceactivator.com/workorders/<workOrderId>` |
| Invoice | `https://app.myserviceactivator.com/invoices/<invoiceId>` |
| Property | `https://app.myserviceactivator.com/properties` (the web app has no page for a single property) |

- Use the `id` returned by the create or update call; never guess or build an ID.
- When one request creates several records (e.g. a customer and a contact during job creation), give a link for each.
- If the call failed, give no link.
