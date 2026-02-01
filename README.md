# Aevoy - Your AI Employee

Email it. It does it.

Aevoy is an email-first AI assistant that actually does tasks for you—books reservations, fills forms, researches topics, and emails you back with results.

## Architecture

```
User emails yourname@aevoy.com
         ↓
Cloudflare Email Worker (catches email)
         ↓
Agent Server (processes task)
   ├── DeepSeek AI (generates response)
   ├── Playwright (browser automation)
   └── Memory System (encrypted storage)
         ↓
Resend API (sends response email)
         ↓
User receives results
```

## Tech Stack

| Component | Technology |
|-----------|------------|
| Frontend | Next.js 14, Tailwind CSS, shadcn/ui |
| Backend | Express.js, TypeScript |
| Database | Supabase (PostgreSQL) |
| Auth | Supabase Auth |
| Email In | Cloudflare Email Workers |
| Email Out | Resend API |
| AI | DeepSeek API, Claude API (vision) |
| Browser | Playwright |

## Project Structure

```
aevoy/
├── apps/
│   └── web/                 # Next.js web app (Vercel)
│       ├── app/             # App Router pages
│       ├── components/      # UI components
│       └── lib/             # Utilities
│
├── packages/
│   └── agent/               # Agent server (VPS)
│       ├── src/
│       │   ├── services/    # Core services
│       │   │   ├── ai.ts    # DeepSeek/Claude
│       │   │   ├── browser.ts # Playwright
│       │   │   ├── email.ts # Resend
│       │   │   ├── memory.ts # Encrypted storage
│       │   │   └── processor.ts # Task processor
│       │   └── index.ts     # Express server
│       └── workspaces/      # User data (encrypted)
│
└── workers/
    └── email-router/        # Cloudflare Worker
```

## Getting Started

### Prerequisites

- Node.js 20+
- pnpm
- Supabase account
- DeepSeek API key
- Resend API key

### Setup

1. **Clone and install dependencies:**

```bash
cd ~/aevoy
pnpm install
```

2. **Configure environment variables:**

```bash
cp .env.example .env
# Edit .env with your API keys
```

3. **Run database migration:**

Go to your Supabase SQL Editor and run the contents of:
```
apps/web/supabase/migration.sql
```

4. **Start development servers:**

```bash
# In one terminal - web app
pnpm --filter web dev

# In another terminal - agent server
pnpm --filter agent dev
```

## Environment Variables

| Variable | Description |
|----------|-------------|
| `NEXT_PUBLIC_SUPABASE_URL` | Supabase project URL |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Supabase anon key |
| `SUPABASE_SERVICE_ROLE_KEY` | Supabase service role key |
| `DEEPSEEK_API_KEY` | DeepSeek API key |
| `ANTHROPIC_API_KEY` | Claude API key (for vision) |
| `RESEND_API_KEY` | Resend API key |
| `AGENT_WEBHOOK_SECRET` | Webhook secret (generate random) |
| `ENCRYPTION_KEY` | 32-byte hex string for AES-256 |

Generate encryption key:
```bash
openssl rand -hex 32
```

## Deployment

### Web App (Vercel)

```bash
cd apps/web
vercel
```

### Agent Server (Docker)

```bash
# Build and run
docker-compose up -d

# View logs
docker-compose logs -f agent
```

### Agent Server (Manual on VPS)

```bash
cd packages/agent
pnpm install
pnpm build
npx playwright install chromium
NODE_ENV=production node dist/index.js
```

## Usage

1. Sign up at aevoy.com
2. Get your AI email address (e.g., yourname@aevoy.com)
3. Email tasks to your AI
4. Receive results in your inbox

### Example Tasks

- "Research the top 5 CRM tools for small businesses"
- "Book a table at Miku for 2 on Saturday 7pm"
- "Fill out this form with my business info" (attach PDF)
- "Every Monday, send me a summary of tech news"

## Privacy & Security

- User data is encrypted at rest (AES-256-GCM)
- Each user has isolated workspace storage
- Email content is never logged
- No third-party tracking
- GDPR-compliant data deletion

## Development

```bash
# Run all services
pnpm dev

# Type check
pnpm typecheck

# Lint
pnpm lint

# Test agent flow
pnpm --filter agent test:flow
```

## License

MIT
