# API Specifications

## Two APIs

1. **Web API** — Dashboard, auth (Vercel)
2. **Agent API** — Task processing (VPS)

---

## Web API

Base: `https://aevoy.com/api`

### GET /api/user

Get current user's profile.

**Response:**
```json
{
  "id": "uuid",
  "username": "omar",
  "email": "omar@gmail.com",
  "aiEmail": "omar@aevoy.com",
  "subscription": {
    "tier": "pro",
    "messagesUsed": 47,
    "messagesLimit": 500
  }
}
```

### PATCH /api/user

Update profile.

**Request:**
```json
{
  "displayName": "Omar",
  "timezone": "America/Vancouver"
}
```

### GET /api/tasks

List user's tasks.

**Query:** `?limit=20&offset=0&status=completed`

**Response:**
```json
{
  "tasks": [
    {
      "id": "uuid",
      "status": "completed",
      "type": "research",
      "subject": "Research CRM tools",
      "createdAt": "2026-01-31T09:00:00Z",
      "completedAt": "2026-01-31T09:02:30Z"
    }
  ],
  "total": 47
}
```

### GET /api/tasks/:id

Get single task.

### GET /api/scheduled-tasks

List scheduled tasks.

### POST /api/scheduled-tasks

Create scheduled task.

**Request:**
```json
{
  "description": "Send tech news summary",
  "cronExpression": "0 8 * * 1",
  "timezone": "America/Vancouver"
}
```

### DELETE /api/scheduled-tasks/:id

Delete scheduled task.

### DELETE /api/user/data

Delete all user data (GDPR).

---

## Agent API

Base: `https://agent.aevoy.com` (or `localhost:3001` in dev)

### POST /task

Receive task from email worker.

**Headers:**
```
X-Webhook-Secret: <secret>
Content-Type: application/json
```

**Request:**
```json
{
  "userId": "uuid",
  "username": "omar",
  "from": "omar@gmail.com",
  "subject": "Research CRM tools",
  "body": "Find top 5 CRM tools for small business",
  "bodyHtml": "<html>...</html>",
  "attachments": []
}
```

**Response:**
```json
{
  "taskId": "uuid",
  "status": "queued"
}
```

### GET /health

Health check.

**Response:**
```json
{
  "status": "healthy",
  "version": "1.0.0",
  "queueDepth": 2
}
```

---

## Cloudflare Email Worker

Not an HTTP API — triggered by incoming email.

**Pseudocode:**
```javascript
export default {
  async email(message, env) {
    // 1. Parse email
    const to = message.to;
    const from = message.from;
    const subject = message.headers.get('subject');
    const body = await message.text();
    
    // 2. Extract username
    const username = to.split('@')[0];
    
    // 3. Look up user in Supabase
    const user = await getUser(username, env.SUPABASE_URL, env.SUPABASE_KEY);
    
    if (!user) {
      // Send bounce email
      return;
    }
    
    // 4. Check quota
    if (user.messages_used >= user.messages_limit) {
      // Send "over quota" email
      return;
    }
    
    // 5. Forward to agent
    await fetch(env.AGENT_URL + '/task', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-Webhook-Secret': env.WEBHOOK_SECRET
      },
      body: JSON.stringify({
        userId: user.id,
        username,
        from,
        subject,
        body
      })
    });
  }
}
```

---

## Error Responses

All errors follow this format:

```json
{
  "error": "error_code",
  "message": "Human readable message"
}
```

**Error codes:**
- `unauthorized` (401) — Not logged in
- `forbidden` (403) — Not allowed
- `not_found` (404) — Resource doesn't exist
- `over_quota` (402) — Message limit reached
- `rate_limited` (429) — Too many requests
- `internal_error` (500) — Server error
