# System Architecture

## Overview

```
┌────────────────────────────────────────────────────────────┐
│                         USER                                │
│                  (sends email to AI)                        │
└─────────────────────────┬──────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────┐
│                     CLOUDFLARE                              │
│  Email Routing: *@aevoy.com → Email Worker                │
│  Worker parses email, validates user, POSTs to agent       │
└─────────────────────────┬──────────────────────────────────┘
                          │
          ┌───────────────┴───────────────┐
          ▼                               ▼
┌─────────────────────────┐   ┌─────────────────────────────┐
│       VERCEL            │   │      AGENT SERVER           │
│    (apps/web)           │   │    (packages/agent)         │
│                         │   │                             │
│  • Landing page         │   │  • Receives tasks           │
│  • Auth (Supabase)      │   │  • AI brain (DeepSeek)      │
│  • Dashboard            │   │  • Browser (Playwright)     │
│  • Activity log         │   │  • Memory system            │
│  • Settings             │   │  • Email sender (Resend)    │
└───────────┬─────────────┘   └──────────────┬──────────────┘
            │                                 │
            └────────────┬────────────────────┘
                         ▼
┌────────────────────────────────────────────────────────────┐
│                      SUPABASE                               │
│  • PostgreSQL database                                      │
│  • Authentication                                           │
│  • File storage                                             │
└────────────────────────────────────────────────────────────┘
```

## Components

### 1. Cloudflare Email Worker

Receives all emails to `*@aevoy.com`.

```
Email arrives → Parse (from, to, subject, body) → 
  Extract username from "to" → 
    Validate user exists →
      POST to agent server
```

### 2. Web App (Vercel)

Next.js 14 application.

**Pages:**
- `/` — Landing page (marketing)
- `/login` — Login
- `/signup` — Sign up
- `/dashboard` — Main dashboard (shows AI email, recent activity)
- `/dashboard/activity` — Full activity log
- `/dashboard/settings` — User settings

### 3. Agent Server

Node.js + Express server that does the actual work.

**Flow:**
```
1. Receive task from email worker
2. Create task record in database
3. Load user's memory from disk
4. Build prompt with memory + task
5. Send to DeepSeek API
6. Parse AI response for actions
7. Execute actions (browse, screenshot, etc.)
8. Update memory with new learnings
9. Send response email via Resend
10. Update task as completed
```

### 4. Memory System

Each user has isolated storage:

```
workspaces/
└── {userId}/
    ├── MEMORY.md       # Long-term facts
    ├── memory/
    │   └── 2026-01-31.md  # Daily log
    └── files/          # User's files
```

**MEMORY.md example:**
```markdown
# About User
- Name: Omar
- Location: Vancouver, BC
- Business: Law firm

# Preferences
- Prefers concise responses
- Timezone: PST

# Learned
- Likes Miku restaurant
- Works on AI automation
```

### 5. AI Brain

Uses DeepSeek API for most tasks (cheap).
Falls back to Claude for complex reasoning or vision.

**Prompt structure:**
```
SYSTEM: You are {username}'s AI assistant. You can DO things.

MEMORY:
{contents of MEMORY.md}

RECENT:
{today's log}

ACTIONS AVAILABLE:
- browse(url): Open webpage
- search(query): Web search
- screenshot(url): Take screenshot
- fill_form(url, fields): Fill form
- send_email(to, subject, body): Send email
- remember(fact): Save to memory
- schedule(task, when): Schedule task

USER'S REQUEST:
{email content}

Respond with your plan, then execute.
```

### 6. Browser Automation

Playwright for web interaction.

**Capabilities:**
- Navigate to URLs
- Fill forms
- Click buttons
- Take screenshots
- Download files
- Solve CAPTCHAs (via Claude Vision)

## Data Flow: Example Task

**User sends:** "Book table at Miku for Saturday 7pm"

```
1. Email arrives at omar@aevoy.com
2. Cloudflare Worker:
   - Parses email
   - Looks up user "omar" in Supabase
   - POSTs to agent server

3. Agent Server:
   - Creates task record
   - Loads omar's memory (knows he's in Vancouver)
   - Sends to DeepSeek: "Book table at Miku Vancouver..."
   
4. DeepSeek responds:
   - "I'll open Miku's website and book"
   - Action: browse("https://mukulrestaurant.com/reservations")

5. Agent executes:
   - Playwright opens page
   - Fills: Party size = 2, Date = Saturday, Time = 7pm
   - Encounters CAPTCHA
   
6. CAPTCHA solving:
   - Screenshot CAPTCHA
   - Send to Claude Vision
   - Get solution
   - Enter solution
   
7. Complete booking:
   - Submit form
   - Screenshot confirmation
   - Save to memory: "Omar likes Miku"
   
8. Send response:
   - Resend API: Email to omar@gmail.com
   - Subject: "Re: Book table at Miku"
   - Body: "Done! Confirmation #12345"
   - Attachment: screenshot.png

9. Update database:
   - Task status = completed
   - Increment usage counter
```

## Deployment

### Development
- Web: `localhost:3000`
- Agent: `localhost:3001`
- Database: Supabase cloud (free tier)

### Production
- Web: Vercel (auto-deploy from GitHub)
- Agent: Hetzner VPS (€4.50/month)
- Database: Supabase (free tier)
- Email: Cloudflare (free) + Resend (free tier)
