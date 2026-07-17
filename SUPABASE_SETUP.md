# Supabase Setup — Registration Form

The registration form on the site saves real submissions to a Supabase table.
Follow these steps once and the form goes live.

> **Shared project note:** This Supabase project is shared with other websites,
> so this event uses its own **namespaced table** `mc2026_registrations`
> (not a generic `registrations`). That keeps these signups completely isolated
> from every other site in the same project — no name clashes, no mixed data.

## 1. Use your existing project (or create one)
Any Supabase project works. If you're reusing a project that already powers
other sites, that's fine — the namespaced table below won't touch them.

## 2. Create the `mc2026_registrations` table
Open **SQL Editor** in the Supabase dashboard, paste this, and click **Run**:

```sql
-- Registrations for Mission Conference 2026 (namespaced for a shared project)
create table if not exists public.mc2026_registrations (
  id          uuid primary key default gen_random_uuid(),
  name        text not null,
  email       text not null,
  phone       text not null,
  role        text not null,
  created_at  timestamptz not null default now()
);

-- Prevent the same email registering twice (for THIS event only)
create unique index if not exists mc2026_registrations_email_unique
  on public.mc2026_registrations (lower(email));

-- Turn on Row Level Security
alter table public.mc2026_registrations enable row level security;

-- Allow anonymous visitors to INSERT (register) — but NOT read others' data
create policy "mc2026 public can register"
  on public.mc2026_registrations
  for insert
  to anon
  with check (true);
```

> This lets the public **insert** registrations but never **read** the list,
> so submitted emails/phones stay private. You (the project owner) can always
> view them in the dashboard under **Table Editor → mc2026_registrations**.
> Because RLS + the policy are scoped to this one table, none of your other
> sites in the project are affected.

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
- **Table Editor → mc2026_registrations** to browse.
- **... → Export to CSV** to download the guest list.

---

### Notes
- The **anon key is meant to be public.** Security comes from Row Level
  Security (the policy above), not from hiding the key.
- The unique index means a person can register only once per email — the form
  shows a friendly "already registered" message on a duplicate.
- Want email notifications on each signup? Add a Supabase **Database Webhook**
  or **Edge Function** on `mc2026_registrations` insert.
