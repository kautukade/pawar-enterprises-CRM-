# Pawar Enterprises Advanced CRM

A lightweight advanced CRM built specifically for Pawar Enterprises. It is designed to run as a static web app and currently persists data in the browser using `localStorage`.

## Main Modules

- Dashboard with open leads, pipeline value, collected revenue and pending receivables
- Lead management with CRUD, filters, call and WhatsApp actions
- Sales pipeline with New → Contacted → Qualified → Proposal → Won/Lost stages
- Client directory
- Follow-up queue with upcoming reminders
- Quotation register and link to Pawar Enterprises quotation generator
- Project tracker with status, progress, value, due dates and site manager
- Payment tracker for advance, milestone and final payments
- Task center with due dates, priorities and completion state
- Team performance cards with pipeline vs target
- Reports with opportunity mix, conversion and exportable CSV
- Global search
- Dark mode
- JSON backup/import
- Responsive mobile sidebar

## Existing Pawar Tools Linked

- Main website: https://pawar-enterprises.onrender.com
- Letter pad: https://pawar-enterprises-letterpad.onrender.com
- Quotation generator: https://pawar-enterprises.onrender.com/quotation.html
- Live work page: https://pawar-enterprises.onrender.com/work.html

## Current Data Mode

This version stores CRM data in the current browser under:

`pe_crm_db_v1`

That makes the demo immediately usable without a backend, but data is not shared between different phones/laptops/browsers.

For real production use, connect Supabase using `SUPABASE_SETUP.md` so that:

- Admin/team members can securely log in
- Data is shared across devices
- Roles and permissions are enforced
- Leads, projects, payments and tasks sync in real time
- Audit history can be retained

## Run Locally

Open `index.html` directly, or serve the folder using VS Code Live Server.

## Deployment

The project can be deployed as a static site on Render, Netlify or Vercel.

No build command is required. Publish the repository root (`.`).

## Important Production Note

Do not rely on browser-only storage for live customer, payment or employee records. Before real use with multiple users, enable authentication, database backups and Row Level Security.
