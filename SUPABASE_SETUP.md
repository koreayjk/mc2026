# Supabase Setup — Registration Form

The registration form on the site saves real submissions to a Supabase table.
Follow these steps once and the form goes live.

## 1. Create a project
1. Go to <https://supabase.com> → **New project** (free tier is fine).
2. Pick a name (e.g. `mission-conference-2026`) and a database password.

## 2. Create the `registrations` table
Open **SQL Editor** in the Supabase dashboard, paste this, and click **Run**:

```sql
-- Registrations for Mission Conference 2026
create table if not exists public.registrations (
  id          uuid primary key default gen_random_uuid(),
  name        text not null,
  email       text not null,
  phone       text not null,
  role        text not null,
  created_at  timestamptz not null default now()
);

-- Prevent the same email registering twice
create unique index if not exists registrations_email_unique
  on public.registrations (lower(email));

-- Turn on Row Level Security
alter table public.registrations enable row level security;

-- Allow anonymous visitors to INSERT (register) — but NOT read others' data
create policy "public can register"
  on public.registrations
  for insert
  to anon
  with check (true);
```

> This lets the public **insert** registrations but never **read** the list,
> so submitted emails/phones stay private. You (the project owner) can always
> view them in the dashboard under **Table Editor → registrations**.

## 3. Get your keys
Go to **Project Settings → API** and copy:
- **Project URL** (looks like `https://abcdxyz.supabase.co`)
- **anon public** key (a long JWT — safe to expose in client code)

## 4. Paste them into `index.html`
Near the bottom of `index.html`, find the config block and replace the two values:

```js
const SUPABASE_URL = "https://YOUR-PROJECT-ref.supabase.co";  // ← your Project URL
const SUPABASE_ANON_KEY = "YOUR-PUBLIC-ANON-KEY";              // ← your anon public key
```

Commit and push — the form is now live and saving to Supabase.

## 5. View / export registrations
- **Table Editor → registrations** to browse.
- **... → Export to CSV** to download the guest list.

---

### Notes
- The **anon key is meant to be public.** Security comes from Row Level
  Security (the policy above), not from hiding the key.
- The unique index means a person can register only once per email — the form
  shows a friendly "already registered" message on a duplicate.
- Want email notifications on each signup? Add a Supabase **Database Webhook**
  or **Edge Function** on `registrations` insert.
