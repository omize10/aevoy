# Product Requirements Document (PRD)

## Product

**Aevoy** — Your AI Employee

**Tagline:** "Email it. It does it."

## Problem

ChatGPT and Claude are just chatbots:
- You go to them (they never reach out)
- They only talk, don't actually DO things
- They forget everything
- Can't work autonomously

## Solution

An AI that:
- You email like a human assistant
- Actually DOES tasks (books, researches, fills forms)
- Remembers everything forever
- Works 24/7 without you
- Contacts YOU when needed

## Target Users

1. **Busy professionals** — Need assistant, can't afford human
2. **Small business owners** — Repetitive tasks pile up
3. **Non-technical people** — Want AI without learning apps

## Core Features (MVP)

### 1. Sign Up & Get AI Email
- Sign up with email/password
- Instantly get: `yourname@aevoy.com`
- See dashboard with your AI's activity

### 2. Email Your AI
- Send email to your AI address
- AI processes and responds
- Attachments supported (PDFs, images)

### 3. AI Actually Does Things
- **Research:** Searches web, compiles reports
- **Writing:** Drafts emails, documents
- **Forms:** Fills out forms on websites
- **Booking:** Makes reservations
- **Monitoring:** Watches for changes, alerts you

### 4. Memory
- Remembers all conversations
- Learns your preferences
- Knows your context (location, business, etc.)

### 5. Proactive Actions
- Scheduled tasks ("Send me news every Monday")
- Alerts ("Tell me if competitor updates site")
- Reminders

## User Stories

### Research
```
Email: "Find top 5 project management tools for small teams"
Response: Detailed comparison with pricing, pros/cons, recommendation
```

### Booking
```
Email: "Book dinner at Miku for 2, Saturday 7pm"
Response: "Done! Confirmation #12345" + screenshot
```

### Form Filling
```
Email: [PDF attachment] "Fill this with my business info"
Response: Completed PDF attached
```

### Scheduled Report
```
Email: "Every Monday 8am, send me a summary of tech news"
Response: "Scheduled! First report coming Monday."
(Then automatic emails every Monday)
```

## Pricing

| Tier | Price | Messages |
|------|-------|----------|
| Free Trial | $0 | 20 total |
| Starter | $9/mo | 100/mo |
| Pro | $19/mo | 500/mo |
| Business | $49/mo | 2000/mo |

## Success Metrics

**Week 1:** 10 beta users, 80% task success rate
**Month 1:** 100 users, 5% conversion to paid
**Month 3:** 1000 users, $5K MRR

## What's Different from ChatGPT

| Feature | ChatGPT | Aevoy |
|---------|---------|-------|
| Email interface | ❌ | ✅ |
| Does real tasks | ❌ | ✅ |
| Messages you first | ❌ | ✅ |
| Unlimited memory | ❌ | ✅ |
| Works while you sleep | ❌ | ✅ |
| Solves CAPTCHAs | ❌ | ✅ |

## Timeline

- Day 1-2: Auth, database, basic UI
- Day 3-4: Agent core, email integration
- Day 5-6: Browser automation
- Day 7: Polish, deploy
