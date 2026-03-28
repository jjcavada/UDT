# UDT — United Defense Tactical

System architecture and connection map for United Defense Tactical franchise across 3 locations: **Grapevine** (TX), **Scottsdale** (AZ), and **Glendale** (AZ).

## What's Inside

- **`UDT_System_MindMap.html`** — Interactive system architecture dashboard showing all Retell AI agents, n8n workflows, GoHighLevel CRM connections, and their current status across all 3 locations. Open in any browser.

## Tech Stack

| Platform | Purpose |
|----------|---------|
| Retell AI | 7 voice AI agents (outbound, inbound, reactivation) |
| n8n | 12 automation workflows (call triggers, post-call analysis, follow-up cadence) |
| GoHighLevel | CRM with 3 sub-accounts (contacts, calendars, pipelines) |
| Chat-Dash | Alternative backend for 2 agents (Scottsdale outbound, Grapevine inbound) |

## Locations

- **Grapevine** — Fully operational (gold standard)
- **Scottsdale** — Operational with 1 issue (inbound agent missing webhook)
- **Glendale** — 6 issues pending (workflows cloned from Grapevine, configs not updated)

## Last Verified

March 28, 2026 — Full audit via Retell API + n8n API
