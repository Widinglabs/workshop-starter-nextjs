---
agent: 'agent'
description: 'Interactive setup guide - install dependencies, configure env, start server'
---

# Quickstart Setup Guide

Interactive setup guide - install dependencies, configure environment, start server.

Think through each step carefully to ensure nothing is missed.

---

## Phase 1: CHECK - Current State

### 1.1 Check Prerequisites

Run these commands to understand current state:

```bash
# Check Node/Bun
bun --version

# Check if dependencies installed
ls node_modules 2>/dev/null || echo "NOT_INSTALLED"

# Check if .env.local exists
ls .env.local 2>/dev/null || echo "NOT_CONFIGURED"
```

**Report findings to user:**
- Bun version: {version}
- Dependencies: Installed / Not installed
- Environment: Configured / Not configured

---

## Phase 2: INSTALL - Dependencies

### 2.1 Install Dependencies

```bash
bun install
```

**If errors occur**, report them and suggest fixes.

---

## Phase 3: CONFIGURE - Environment Variables

### 3.1 Check Existing Configuration

```bash
cat .env.local 2>/dev/null || echo "No .env.local found"
```

### 3.2 Explain Required Variables

**Ask the user:**

> **Environment Configuration**
>
> This project requires Supabase for authentication and database.
>
> **MANDATORY** (app won't work without these):
>
> | Variable | Description | Where to get it |
> |----------|-------------|-----------------|
> | `NEXT_PUBLIC_SUPABASE_URL` | Your Supabase project URL | Supabase Dashboard → Settings → API |
> | `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Public anon key | Supabase Dashboard → Settings → API |
> | `DATABASE_URL` | PostgreSQL connection string | Supabase Dashboard → Settings → Database → Connection string (use Transaction pooler, port 6543) |
>
> **OPTIONAL** (for additional features):
>
> | Variable | Description | Default |
> |----------|-------------|---------|
> | `LOG_LEVEL` | Logging verbosity | `info` |
> | `NODE_ENV` | Environment mode | `development` |
>
> **Do you have a Supabase project set up?**
> - Yes, I have the credentials ready
> - No, I need to create one first
> - Skip for now (I'll configure later)

**GATE**: Wait for user response.

### 3.3 Guide Based on Response

**If "Yes, I have credentials":**

Ask for:
1. **Supabase URL** (looks like `https://xxxxx.supabase.co`)
2. **Anon Key** (starts with `eyJ...`)
3. **Database URL** (looks like `postgresql://postgres.[ref]:[password]@...`)

Then create `.env.local` with the provided values.

**If "No, I need to create one":**

Guide them:
1. Go to https://supabase.com and sign up/login
2. Click "New Project"
3. Choose organization, name project, set database password
4. Wait for project to provision (~2 minutes)
5. Go to Settings → API to get URL and anon key
6. Go to Settings → Database → Connection string for DATABASE_URL
   - Use "Transaction pooler" mode (port 6543)
   - Replace `[YOUR-PASSWORD]` with database password

**If "Skip for now":**

Continue but warn that app will fail to connect to Supabase.

---

## Phase 4: DATABASE - Setup (if configured)

### 4.1 Run Migrations (Optional)

**Ask the user:**

> Would you like to push the database schema to Supabase?
> ⚠️ This will create/update tables in your database.

**If yes:**

```bash
bun run db:push
```

### 4.2 Auth Trigger Setup

Inform user they need to run this SQL in Supabase Dashboard → SQL Editor:

```sql
-- Function to sync auth.users to public.users
CREATE OR REPLACE FUNCTION public.handle_new_user()
RETURNS trigger AS $$
BEGIN
  INSERT INTO public.users (id, email)
  VALUES (NEW.id, NEW.email);
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

-- Trigger on auth.users insert
CREATE OR REPLACE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE FUNCTION public.handle_new_user();
```

---

## Phase 5: VALIDATE - Check Setup

```bash
# Type check
npx tsc --noEmit

# Lint check
bun run lint
```

Report any errors and suggest fixes.

---

## Phase 6: START - Run Development Server

```bash
bun run dev
```

Wait for server to start, then verify it's running at http://localhost:3000

---

## Phase 7: REPORT - Summary

Output a summary:

```markdown
## Quickstart Complete

### What Was Done

- [x] Dependencies installed via `bun install`
- [x] Environment configured in `.env.local`
- [x] Database schema pushed (if selected)
- [x] Validation checks passed
- [x] Development server started

### Access Points

| Service | URL |
|---------|-----|
| Frontend | http://localhost:3000 |

### Next Steps

1. **Open the app**: http://localhost:3000
2. **Register a user**: Click "Sign Up" to test auth
3. **Start coding**: Edit `src/app/page.tsx` to see hot reload

### Useful Commands

| Command | Description |
|---------|-------------|
| `bun run dev` | Start dev server |
| `bun run build` | Production build |
| `bun run lint` | Check for lint errors |
| `bun test` | Run tests |
| `bun run db:studio` | Open Drizzle Studio |

### Common Issues

| Error | Fix |
|-------|-----|
| Port 3000 in use | `lsof -ti:3000 | xargs kill` |
| Database connection failed | Check DATABASE_URL in `.env.local` |
| Auth not working | Run the SQL trigger in Supabase |
```
