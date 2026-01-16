---
description: Interactive setup guide - install dependencies, configure env, start server
---

# Quickstart Setup Guide

Think through each step carefully to ensure nothing is missed.

---

## Phase 1: CHECK - Current State

### 1.1 Check Prerequisites

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

> Great! Please provide your Supabase credentials:
>
> 1. **Supabase URL** (looks like `https://xxxxx.supabase.co`):
> 2. **Anon Key** (starts with `eyJ...`):
> 3. **Database URL** (looks like `postgresql://postgres.[ref]:[password]@...`):
>
> I'll create your `.env.local` file.

**GATE**: Wait for credentials, then create `.env.local`:

```bash
cat > .env.local << 'EOF'
# Supabase Configuration
NEXT_PUBLIC_SUPABASE_URL={provided_url}
NEXT_PUBLIC_SUPABASE_ANON_KEY={provided_key}
DATABASE_URL={provided_database_url}

# Optional
LOG_LEVEL=info
NODE_ENV=development
EOF
```

**If "No, I need to create one":**

> **Quick Supabase Setup:**
>
> 1. Go to https://supabase.com and sign up/login
> 2. Click "New Project"
> 3. Choose organization, name your project, set a database password
> 4. Wait for project to provision (~2 minutes)
> 5. Go to Settings → API to get your URL and anon key
> 6. Go to Settings → Database → Connection string for DATABASE_URL
>    - Use "Transaction pooler" mode (port 6543)
>    - Replace `[YOUR-PASSWORD]` with your database password
>
> **When ready, run `/quickstart` again with your credentials.**

**STOP here if user needs to create Supabase project.**

**If "Skip for now":**

> Skipping environment setup. Note: The app will fail to connect to Supabase.
> Run `/quickstart` again when you have your credentials ready.

**Continue to Phase 4 but warn about missing config.**

### 3.4 Optional Configuration

**Ask the user:**

> **Optional Settings**
>
> Would you like to configure any optional settings?
>
> - [ ] **LOG_LEVEL** - Set logging verbosity (debug/info/warn/error)
> - [ ] **Skip** - Use defaults
>
> (Select what you'd like to configure)

**GATE**: If user wants to configure, update `.env.local` accordingly.

---

## Phase 4: DATABASE - Setup (if configured)

### 4.1 Check Database Connection

**Only if DATABASE_URL is configured:**

```bash
# Test database connection by running type check (will fail if schema can't connect)
npx tsc --noEmit 2>&1 | head -20
```

### 4.2 Run Migrations (Optional)

**Ask the user:**

> **Database Setup**
>
> Would you like to push the database schema to Supabase?
>
> ⚠️ This will create/update tables in your database.
>
> - Yes, push schema now
> - No, I'll do it later

**If "Yes":**

```bash
bun run db:push
```

### 4.3 Setup Auth Trigger

**Inform the user:**

> **Important**: For user authentication to work, you need to run this SQL in your Supabase Dashboard → SQL Editor:
>
> ```sql
> -- Function to sync auth.users to public.users
> CREATE OR REPLACE FUNCTION public.handle_new_user()
> RETURNS trigger AS $$
> BEGIN
>   INSERT INTO public.users (id, email)
>   VALUES (NEW.id, NEW.email);
>   RETURN NEW;
> END;
> $$ LANGUAGE plpgsql SECURITY DEFINER;
>
> -- Trigger on auth.users insert
> CREATE OR REPLACE TRIGGER on_auth_user_created
>   AFTER INSERT ON auth.users
>   FOR EACH ROW EXECUTE FUNCTION public.handle_new_user();
> ```
>
> Have you run this SQL, or would you like me to remind you later?

---

## Phase 5: VALIDATE - Check Setup

### 5.1 Run Validation

```bash
# Type check
npx tsc --noEmit

# Lint check
bun run lint
```

**Report any errors and suggest fixes.**

---

## Phase 6: START - Run Development Server

### 6.1 Start Server

```bash
# Start dev server in background
bun run dev &

# Wait for server to start
sleep 3

# Check if running
curl -s http://localhost:3000 > /dev/null && echo "SERVER_RUNNING" || echo "SERVER_FAILED"
```

### 6.2 Verify Server

```bash
# Get server status
lsof -i :3000 | grep LISTEN
```

---

## Phase 7: REPORT - Summary

**Output to user:**

```markdown
## 🚀 Quickstart Complete

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
| Supabase Dashboard | {SUPABASE_URL} |

### Configuration Summary

| Setting | Value |
|---------|-------|
| Supabase URL | `{masked_url}` |
| Database | Connected / Not configured |
| Log Level | {LOG_LEVEL} |

### Next Steps

1. **Open the app**: http://localhost:3000
2. **Register a user**: Click "Sign Up" to test auth
3. **Check the dashboard**: Login to see protected routes
4. **Start coding**: Edit `src/app/page.tsx` to see hot reload

### Useful Commands

| Command | Description |
|---------|-------------|
| `bun run dev` | Start dev server |
| `bun run build` | Production build |
| `bun run lint` | Check for lint errors |
| `bun test` | Run tests |
| `bun run db:studio` | Open Drizzle Studio |

### Issues?

- **Port 3000 in use**: Kill the process with `lsof -ti:3000 | xargs kill`
- **Database connection failed**: Check your DATABASE_URL in `.env.local`
- **Auth not working**: Make sure you ran the SQL trigger in Supabase

Happy coding! 🎉
```

---

## Error Handling

**If any step fails:**

1. Report the specific error
2. Suggest common fixes
3. Ask if user wants to continue or abort
4. Log the issue for the final report

**Common Issues:**

| Error | Likely Cause | Fix |
|-------|--------------|-----|
| `bun: command not found` | Bun not installed | `curl -fsSL https://bun.sh/install | bash` |
| `ECONNREFUSED` on database | Wrong DATABASE_URL | Check connection string format |
| `Invalid API key` | Wrong ANON_KEY | Copy fresh key from Supabase Dashboard |
| Port 3000 in use | Another process | `lsof -ti:3000 | xargs kill` |
