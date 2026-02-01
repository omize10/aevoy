# CLAUDE.md — Instructions for Claude Code

## What You Are Building

**Aevoy** — An email-first AI assistant service.

Users sign up, get an AI email address (e.g., `omar@aevoy.com`), and email tasks to their AI. The AI actually DOES things (browses web, fills forms, books reservations) and emails back results.

## Core Architecture

```
User emails omar@aevoy.com
       ↓
Cloudflare Email Worker catches it
       ↓
Worker POSTs to Agent Server
       ↓
Agent Server:
  1. Loads user's memory
  2. Sends task to DeepSeek API
  3. Executes actions (Playwright browser)
  4. Updates memory
  5. Sends response via Resend
       ↓
User receives email with results
```

## Tech Stack

| Component | Technology |
|-----------|------------|
| Frontend | Next.js 14 (App Router), Tailwind, shadcn/ui |
| Backend | Node.js + TypeScript + Express |
| Database | Supabase (PostgreSQL) |
| Auth | Supabase Auth |
| Email In | Cloudflare Email Workers |
| Email Out | Resend API |
| AI | DeepSeek API (cheap), Claude API (complex tasks) |
| Browser | Playwright |
| Web Host | Vercel |
| Agent Host | Hetzner VPS (later) / Local (dev) |

## Project Structure

```
aevoy/
├── CLAUDE.md                 # This file
├── .env.example              # Environment template
├── package.json              # Root package.json (pnpm workspace)
├── pnpm-workspace.yaml
│
├── docs/                     # Documentation
│   ├── PRD.md               # Product requirements
│   ├── ARCHITECTURE.md      # System design
│   ├── DATABASE.md          # Database schema
│   ├── API.md               # API specs
│   └── PRIVACY.md           # Privacy implementation
│
├── apps/
│   └── web/                 # Next.js app (Vercel)
│       ├── app/
│       │   ├── page.tsx           # Landing page
│       │   ├── (auth)/
│       │   │   ├── login/page.tsx
│       │   │   └── signup/page.tsx
│       │   ├── dashboard/
│       │   │   ├── page.tsx       # Main dashboard
│       │   │   ├── activity/page.tsx
│       │   │   └── settings/page.tsx
│       │   └── api/
│       │       ├── tasks/route.ts
│       │       └── webhooks/
│       ├── components/
│       ├── lib/
│       │   ├── supabase/
│       │   └── utils.ts
│       └── package.json
│
├── packages/
│   └── agent/               # Agent server
│       ├── src/
│       │   ├── index.ts           # Express entry
│       │   ├── routes/
│       │   │   └── task.ts        # POST /task
│       │   ├── services/
│       │   │   ├── ai.ts          # DeepSeek/Claude calls
│       │   │   ├── browser.ts     # Playwright
│       │   │   ├── memory.ts      # Memory system
│       │   │   └── email.ts       # Resend
│       │   ├── workers/
│       │   │   └── processor.ts   # Task queue processor
│       │   └── types/
│       ├── workspaces/            # User data (gitignored)
│       └── package.json
│
└── workers/
    └── email-router/        # Cloudflare Worker
        ├── src/
        │   └── index.ts
        ├── wrangler.toml
        └── package.json
```

## Development Phases

### Phase 1: Foundation (Day 1)
1. Initialize monorepo with pnpm
2. Set up Next.js app with Supabase auth
3. Create landing page
4. Create login/signup pages
5. Create basic dashboard

### Phase 2: Database (Day 1-2)
1. Create Supabase project
2. Run database migrations (profiles, tasks tables)
3. Set up Row Level Security
4. Test auth flow

### Phase 3: Agent Core (Day 2-3)
1. Create Express server
2. Implement memory system
3. Integrate DeepSeek API
4. Create task processor
5. Test with mock emails

### Phase 4: Email Integration (Day 3-4)
1. Set up Cloudflare Email Worker
2. Connect worker to agent
3. Set up Resend for sending
4. Test full email flow

### Phase 5: Browser Automation (Day 4-5)
1. Set up Playwright
2. Implement basic browsing
3. Add screenshot capability
4. Add form filling
5. Add CAPTCHA solving (Claude Vision)

### Phase 6: Polish (Day 5-7)
1. Activity feed in dashboard
2. Usage tracking
3. Error handling
4. Testing
5. Deploy to production

## Key Commands

```bash
# Install all dependencies
pnpm install

# Run web app (development)
pnpm --filter web dev

# Run agent server (development)
pnpm --filter agent dev

# Run both
pnpm dev

# Build for production
pnpm build

# Deploy web to Vercel
cd apps/web && vercel

# Type check
pnpm typecheck

# Lint
pnpm lint
```

## Environment Variables

Required in `.env`:

```bash
# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=xxx
SUPABASE_SERVICE_ROLE_KEY=xxx

# AI
DEEPSEEK_API_KEY=xxx
ANTHROPIC_API_KEY=xxx  # For vision/complex tasks

# Email
RESEND_API_KEY=xxx

# Agent
AGENT_WEBHOOK_SECRET=xxx  # Random string, shared with email worker
AGENT_PORT=3001

# Encryption
ENCRYPTION_KEY=xxx  # 32 byte hex string for AES-256
```

## Coding Standards

1. **TypeScript everywhere** — No `any` types
2. **Async/await** — No callbacks
3. **Error handling** — Always try/catch, never crash
4. **Logging** — Log events, NEVER log user content
5. **Security** — Validate all inputs, parameterized queries
6. **Testing** — Test critical paths

## Security Rules (IMPORTANT)

1. **NEVER** log email content or user messages
2. **NEVER** store API keys in code
3. **ALWAYS** validate user owns the resource
4. **ALWAYS** sanitize inputs before using in prompts
5. **ALWAYS** encrypt user memory files
6. **NEVER** share data between users

## File Reading Order

When starting work, read these files first:
1. `docs/PRD.md` — What we're building
2. `docs/ARCHITECTURE.md` — How it works
3. `docs/DATABASE.md` — Database schema
4. `docs/PRIVACY.md` — Privacy requirements

## When You're Stuck

1. Check the docs/ folder
2. Look at error messages carefully
3. Ask for clarification — don't guess
4. Check similar open-source projects for patterns

## Current Status

Starting fresh. Begin with Phase 1.
