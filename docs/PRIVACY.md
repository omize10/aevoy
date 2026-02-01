# Privacy Implementation

## Core Principle

**User data is sacred. We never compromise on privacy.**

## Privacy Requirements

### 1. Data Isolation

Every user's data is completely separate.

```
workspaces/
├── user-abc/     # User A's data
│   ├── MEMORY.md
│   └── memory/
├── user-xyz/     # User B's data
│   ├── MEMORY.md
│   └── memory/
```

**Implementation:**
- Workspace path: `workspaces/{userId}/`
- User ID from Supabase auth (UUID)
- NEVER access another user's workspace
- Validate user ID on every operation

### 2. No Content Logging

We log events, NOT content.

**✅ DO log:**
```javascript
logger.info('Task started', { 
  taskId: 'uuid',
  userId: 'uuid',
  type: 'research',
  timestamp: new Date()
});
```

**❌ DO NOT log:**
```javascript
// NEVER DO THIS
logger.info('Task content', { 
  emailBody: userEmail,  // NO!
  response: aiResponse   // NO!
});
```

### 3. Encryption at Rest

User memory files are encrypted with AES-256.

```typescript
import crypto from 'crypto';

const ALGORITHM = 'aes-256-gcm';
const KEY = Buffer.from(process.env.ENCRYPTION_KEY, 'hex'); // 32 bytes

export function encrypt(text: string): string {
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv(ALGORITHM, KEY, iv);
  
  let encrypted = cipher.update(text, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  
  const authTag = cipher.getAuthTag();
  
  return iv.toString('hex') + ':' + authTag.toString('hex') + ':' + encrypted;
}

export function decrypt(encryptedData: string): string {
  const [ivHex, authTagHex, encrypted] = encryptedData.split(':');
  
  const iv = Buffer.from(ivHex, 'hex');
  const authTag = Buffer.from(authTagHex, 'hex');
  const decipher = crypto.createDecipheriv(ALGORITHM, KEY, iv);
  
  decipher.setAuthTag(authTag);
  
  let decrypted = decipher.update(encrypted, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  
  return decrypted;
}
```

### 4. No AI Training

- DeepSeek API: Does not train on API calls
- Claude API: Does not train on API calls
- We include explicit opt-out headers where possible

### 5. User Data Deletion

Users can delete all their data with one click.

```typescript
async function deleteAllUserData(userId: string) {
  // 1. Delete from database
  await supabase.from('tasks').delete().eq('user_id', userId);
  await supabase.from('scheduled_tasks').delete().eq('user_id', userId);
  await supabase.from('profiles').delete().eq('id', userId);
  
  // 2. Delete workspace folder
  const workspacePath = path.join(WORKSPACES_DIR, userId);
  await fs.rm(workspacePath, { recursive: true, force: true });
  
  // 3. Delete auth user
  await supabase.auth.admin.deleteUser(userId);
}
```

### 6. No Third-Party Tracking

- No Google Analytics
- No Facebook Pixel
- No tracking cookies
- Only essential auth cookies

### 7. Transparent Activity Log

Users see exactly what their AI did.

```typescript
// Activity log entry (shown to user)
{
  timestamp: '2026-01-31T09:15:00Z',
  action: 'Searched Google for "CRM tools for small business"',
  result: 'Found 5 results'
}

// NOT the actual content, just the action
```

### 8. Secure Communication

- All API calls over HTTPS
- Webhook secret for email worker → agent
- JWT tokens for auth

## Implementation Checklist

### Code Level

- [ ] Workspace isolation by user ID
- [ ] Encryption functions for memory files
- [ ] Logger that strips content
- [ ] Delete user data function
- [ ] Validate user owns resource on every request

### Infrastructure Level

- [ ] HTTPS everywhere
- [ ] Encrypted database (Supabase does this)
- [ ] Secure environment variables
- [ ] No logs shipped to third parties

### UI Level

- [ ] Privacy policy page
- [ ] "Delete my data" button in settings
- [ ] Activity log (shows actions, not content)
- [ ] Clear explanation of what data we store

## Privacy Policy Summary

For the actual privacy policy page:

```
What we collect:
- Email address (for auth and sending responses)
- Task metadata (timestamps, types, NOT content)
- Memory you build with your AI (encrypted, only you can access)

What we DON'T do:
- We don't sell your data
- We don't use your data for AI training
- We don't share with third parties (except to process tasks)
- We don't track you across the web

Your rights:
- Download all your data
- Delete all your data
- Export your memory
```

## Security Considerations

### Prompt Injection Protection

User emails could contain malicious prompts.

```typescript
// Sanitize user input before including in AI prompt
function sanitizeForPrompt(text: string): string {
  // Remove potential injection patterns
  return text
    .replace(/ignore previous instructions/gi, '[filtered]')
    .replace(/system:/gi, '[filtered]')
    .replace(/assistant:/gi, '[filtered]');
}
```

### Rate Limiting

Prevent abuse.

```typescript
// Per-user rate limit
const rateLimiter = new Map<string, number>();

function checkRateLimit(userId: string): boolean {
  const now = Date.now();
  const lastRequest = rateLimiter.get(userId) || 0;
  
  if (now - lastRequest < 5000) { // 5 second minimum between requests
    return false;
  }
  
  rateLimiter.set(userId, now);
  return true;
}
```

### Browser Sandbox

Playwright runs in isolated context.

```typescript
// Each task gets fresh browser context
const context = await browser.newContext({
  // No persistent storage
  storageState: undefined,
  // Clear cookies
  // Isolated from other users
});

// After task
await context.close();
```
