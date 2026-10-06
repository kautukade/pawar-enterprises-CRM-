# Pawar Enterprises CRM — Supabase Production Plan

The current CRM is a browser-persistent working demo. This document defines the production backend so the same CRM can safely work across phones, laptops and multiple team members.

## Recommended Architecture

- Frontend: current HTML/CSS/JavaScript CRM
- Authentication: Supabase Auth
- Database: Supabase PostgreSQL
- Realtime: Supabase Realtime for leads, tasks, projects and payments
- File storage: Supabase Storage for project images, client documents and payment receipts
- Security: Row Level Security on every exposed table
- Hosting: Render static site / Netlify / Vercel

## Core Tables

### profiles
- id uuid primary key references auth.users
- full_name text
- phone text
- role text (`owner`, `admin`, `sales`, `site_manager`, `accounts`, `viewer`)
- is_active boolean
- created_at timestamptz

### leads
- id uuid primary key
- name text
- phone text
- email text
- location text
- service text
- source text
- status text
- estimated_value numeric
- priority text
- next_follow_up date
- owner_id uuid references profiles
- notes text
- created_by uuid references profiles
- created_at timestamptz
- updated_at timestamptz

### clients
- id uuid primary key
- name text
- phone text
- email text
- location text
- billing_address text
- notes text
- created_at timestamptz

### quotations
- id uuid primary key
- quote_no text unique
- client_id uuid references clients
- lead_id uuid references leads
- amount numeric
- status text
- quotation_date date
- valid_till date
- snapshot jsonb
- created_by uuid references profiles
- created_at timestamptz

### projects
- id uuid primary key
- title text
- client_id uuid references clients
- service text
- status text
- progress int check (progress between 0 and 100)
- project_value numeric
- start_date date
- due_date date
- site_address text
- manager_id uuid references profiles
- notes text
- created_at timestamptz

### project_updates
- id uuid primary key
- project_id uuid references projects on delete cascade
- update_text text
- progress int
- created_by uuid references profiles
- created_at timestamptz

### project_media
- id uuid primary key
- project_id uuid references projects on delete cascade
- media_type text (`before`, `progress`, `after`, `document`)
- storage_path text
- caption text
- uploaded_by uuid references profiles
- created_at timestamptz

### payments
- id uuid primary key
- client_id uuid references clients
- project_id uuid references projects
- amount numeric
- payment_type text
- status text
- due_date date
- paid_date date
- method text
- reference_no text
- receipt_path text
- notes text
- created_by uuid references profiles
- created_at timestamptz

### tasks
- id uuid primary key
- title text
- related_type text
- related_id uuid
- assigned_to uuid references profiles
- due_date date
- priority text
- status text
- created_by uuid references profiles
- created_at timestamptz

### activities
- id uuid primary key
- entity_type text
- entity_id uuid
- action text
- description text
- actor_id uuid references profiles
- created_at timestamptz

### notifications
- id uuid primary key
- user_id uuid references profiles
- title text
- body text
- type text
- read_at timestamptz
- created_at timestamptz

## Roles / Permissions

### Owner
Full access to all modules, users, settings, finance and reports.

### Admin
Full operational access except owner-only security settings.

### Sales
Manage assigned leads, follow-ups, clients and quotations. Read projects relevant to their leads/clients.

### Site Manager
Read assigned projects, update progress, upload site media, manage project tasks.

### Accounts
Read clients/projects, manage payments, receipts and finance reports.

### Viewer
Read-only access to explicitly allowed modules.

## RLS Strategy

Enable RLS on every table.

Recommended helper function:

```sql
create or replace function public.current_role()
returns text
language sql
stable
security definer
set search_path = public
as $$
  select role from public.profiles where id = auth.uid();
$$;
```

Policy principles:

- owner/admin: CRUD on all business records
- sales: CRUD assigned leads/tasks + read linked clients/quotations
- site_manager: update assigned projects/project_updates/project_media
- accounts: CRUD payments + read clients/projects
- viewer: select only
- never expose service_role key in browser
- role must come from protected profile/app metadata, not editable user metadata

## Storage Buckets

### project-media
Private bucket. Signed URLs for authorized viewing.

Folders:
- `projects/{project_id}/before/`
- `projects/{project_id}/progress/`
- `projects/{project_id}/after/`
- `projects/{project_id}/docs/`

### payment-receipts
Private bucket.

Folders:
- `payments/{payment_id}/`

## Automations

Recommended scheduled jobs / Edge Functions:

1. Every morning: create notifications for today's lead follow-ups.
2. Every morning: flag unpaid payments past due as `overdue`.
3. When quotation becomes approved: optionally create client + project draft.
4. When lead becomes won: create activity + optional client conversion flow.
5. When project hits 100%: mark completion candidate and request final payment check.

## Audit Trail

For production, log important mutations to `activities` or a dedicated `audit_log` table:
- status changes
- value changes
- project progress
- payment edits
- user/role changes
- deletes

## Migration Order

1. Create Supabase project.
2. Create tables and foreign keys.
3. Enable RLS.
4. Create roles/profiles flow.
5. Create Storage buckets and policies.
6. Replace localStorage reads/writes with Supabase queries.
7. Add Supabase Auth login and protected app shell.
8. Add Realtime subscriptions.
9. Migrate demo/exported JSON if required.
10. Test role by role before production use.

## Security Rules

- Publishable/anon key can be used in browser with correct RLS.
- Never put `service_role` in frontend code.
- Do not use hard-coded production passwords.
- Finance and customer records must not be publicly selectable.
- Test RLS with separate test accounts for every role before launch.
