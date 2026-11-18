# Service Activator for Claude

Work with your [Service Activator](https://www.myserviceactivator.com) account from Claude in plain language: find, view, create and update service jobs, work orders, customers and contacts, and look up properties and invoices. You never need to know field names or IDs: Claude asks for names, looks the records up, checks your input the same way the web app does, and asks for your confirmation before it saves anything.

## What is included

- **Skill `service-activator`**: step-by-step instructions that teach Claude how to use the Service Activator tools well (lookups by name, validation rules, confirmation before saving, error handling, links to the web app).
- **MCP server `service-activator`**: the remote Service Activator MCP server at `https://mcp.abren.ai/mcp`, which provides the tools. If you also added the Service Activator connector from the Claude directory, it is the same server, so you see one set of tools.

## Requirements

- An active Service Activator subscription and a user account.
- What you can see and do follows your role in Service Activator. For example, technicians cannot create jobs, and technicians and supervisors cannot list invoices.

## Getting started

1. Install the plugin in Claude.
2. When Claude first uses a Service Activator tool, sign in with your Service Activator account (OAuth, at `https://mcp.myserviceactivator.com`).
3. Ask, for example:
   - "Show me all active jobs for Greenfield Plaza."
   - "Add a new customer Greenfield Plaza with Sarah Lee as the main contact."
   - "Show me the latest invoice for Greenfield Plaza."
   - "What is scheduled at the Main Street property tomorrow?"

## What it can do

| Area | Read | Create / update |
|---|---|---|
| Jobs (one-time or recurring) | Yes | Yes |
| Work orders (visits) | Yes | Yes |
| Customers | Yes | Yes |
| Contacts | Yes | Yes |
| Properties | Yes | Web app only |
| Invoices | Yes | Web app only |
| Employees | Lookup only | — |

Deleting or cancelling records, payments, timesheets and quotes are not available; use the web app at https://app.myserviceactivator.com.

## Data and privacy

- This plugin contains only instructions (Markdown) and the address of the Service Activator MCP server. It runs no scripts, installs no packages and has no hooks.
- All data goes only between Claude and Service Activator's own servers: `https://mcp.abren.ai` (MCP tools) and `https://mcp.myserviceactivator.com` (sign-in). Nothing is sent anywhere else.
- Claude only sees the records your Service Activator account is allowed to see, and only the data needed for the request you make.
- Privacy policy: https://www.myserviceactivator.com/privacy-policy/

## Support

Email support@myserviceactivator.com.

## License

See [LICENSE](LICENSE).
