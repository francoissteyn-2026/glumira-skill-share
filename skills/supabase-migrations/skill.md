---
name: supabase-migrations
description: >
  Schema migration discipline for Supabase projects. Every schema change goes
  through a versioned migration file — never directly in the dashboard. Claude
  generates, reviews, and applies migrations. Covers the full cycle: new table,
  alter column, RLS policy, seed data, rollback. Wired into the session
  closing protocol so no schema change ships without a migration file.
  Trigger phrases: "migration", "supabase migration", "schema change",
  "add column", "new table", "rls policy", "seed data", "db migration".
---

# Supabase Migrations — Schema Versioning for Claude Code

Every schema change that isn't in a migration file doesn't exist.
Dashboard edits are invisible to git, invisible to teammates, and invisible
to the next session that boots without your memory of what you clicked.
This skill enforces the discipline.

**The rule:** no schema change ships without a `supabase/migrations/` file
committed alongside it. Claude generates the SQL, you review it, it runs.

---

## Install Supabase CLI

```bash
npm install -g supabase
```

Or via Scoop (Windows):
```powershell
scoop bucket add supabase https://github.com/supabase/scoop-bucket.git
scoop install supabase
```

Verify:
```bash
supabase --version
```

---

## Project setup

```bash
# In your project root — creates supabase/ folder structure
supabase init

# Link to your remote project
supabase link --project-ref <your-project-ref>

# Pull current remote schema as baseline (first time only)
supabase db pull
```

The `supabase/migrations/` folder is your source of truth. Commit it.

---

## The migration workflow

### 1. Create a new migration

```bash
supabase migration new <descriptive-name>
# Creates: supabase/migrations/YYYYMMDDHHMMSS_descriptive-name.sql
```

**Naming convention:**
```
20260514_120000_add_insulin_doses_table.sql
20260514_130000_add_rls_to_profiles.sql
20260514_140000_alter_users_add_timezone.sql
```

### 2. Claude writes the SQL

Tell Claude: "Write the migration SQL for [schema change]."

Common patterns:

**New table:**
```sql
create table if not exists doses (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references auth.users(id) on delete cascade not null,
  units numeric(5,2) not null,
  insulin_type text not null,
  injected_at timestamptz not null,
  created_at timestamptz default now() not null
);

alter table doses enable row level security;
```

**Add column:**
```sql
alter table profiles
add column if not exists timezone text default 'UTC' not null;
```

**RLS policy:**
```sql
create policy "Users can only access their own doses"
  on doses for all
  using (auth.uid() = user_id)
  with check (auth.uid() = user_id);
```

**Index for performance:**
```sql
create index if not exists idx_doses_user_injected
  on doses(user_id, injected_at desc);
```

### 3. Review before applying

Always read the SQL before running it. A DROP TABLE with no backup is permanent.

```bash
# Preview what will run (dry run)
supabase db diff
```

### 4. Apply locally first

```bash
supabase db reset   # Apply all migrations to local DB from scratch
# OR
supabase migration up  # Apply only new migrations
```

### 5. Apply to remote

```bash
supabase db push
```

---

## Rollback

Supabase doesn't have built-in rollback. The discipline is: write a new
migration that reverses the change.

```bash
supabase migration new revert_add_doses_table
```

```sql
-- supabase/migrations/YYYYMMDDHHMMSS_revert_add_doses_table.sql
drop table if exists doses;
```

This preserves history. Never delete a migration file that has been applied to production.

---

## Seed data

```bash
# supabase/seed.sql — runs after reset, not on production push
```

```sql
-- supabase/seed.sql
insert into insulin_types (name, onset_minutes, peak_minutes, duration_minutes)
values
  ('Humalog', 15, 90, 240),
  ('Levemir', 60, 180, 1200);
```

---

## TypeScript types from schema

After every migration, regenerate types:

```bash
supabase gen types typescript --linked > src/types/database.types.ts
```

Add to your session closing protocol:
```
Before commit: supabase gen types → commit updated database.types.ts
```

---

## Wiring into Claude Code sessions

Add to your `/start` command:

```
Check supabase/migrations/ — confirm last migration matches last schema change.
If there's a schema change in this session, create a migration file before committing.
```

Add to your closing protocol (before option 2 — Summarize/Save):

```
1. supabase gen types → update database.types.ts
2. supabase db push (if migration was created)
3. commit migration file + types together
```

---

## Common mistakes

| Mistake | Cost | Fix |
|---------|------|-----|
| Dashboard edit without migration | Schema drift — next `db reset` loses the change | Always create the migration file first |
| Migration with no RLS | Security hole on every new table | Template: enable RLS + 4 policies as a block |
| Committing app code without migration | Type errors in CI | Migration + types commit together, always |
| Using `drop table` in a reversible migration | Permanent data loss | Use `alter table disable` or archive pattern |

---

## The 4-policy RLS template

Every new table gets these 4 policies. No exceptions.

```sql
-- Enable RLS
alter table <table> enable row level security;

-- SELECT
create policy "<table>_select" on <table>
  for select using (auth.uid() = user_id);

-- INSERT
create policy "<table>_insert" on <table>
  for insert with check (auth.uid() = user_id);

-- UPDATE
create policy "<table>_update" on <table>
  for update using (auth.uid() = user_id)
  with check (auth.uid() = user_id);

-- DELETE
create policy "<table>_delete" on <table>
  for delete using (auth.uid() = user_id);
```
