# Database Schema

Using Supabase (PostgreSQL) with Row Level Security.

## Tables

### profiles

Extends Supabase auth.users with app-specific data.

```sql
CREATE TABLE public.profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  username TEXT UNIQUE NOT NULL,
  email TEXT NOT NULL,
  display_name TEXT,
  
  -- Settings
  timezone TEXT DEFAULT 'America/Los_Angeles',
  
  -- Subscription
  subscription_tier TEXT DEFAULT 'free', -- free, starter, pro, business
  messages_used INT DEFAULT 0,
  messages_limit INT DEFAULT 20,
  
  -- Timestamps
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  last_active_at TIMESTAMPTZ
);

CREATE UNIQUE INDEX idx_profiles_username ON profiles(username);
```

### tasks

Records of every task processed.

```sql
CREATE TABLE public.tasks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  
  -- Task info
  status TEXT DEFAULT 'pending', -- pending, processing, completed, failed
  type TEXT, -- research, booking, form, writing, etc.
  
  -- Email metadata (NOT content)
  email_subject TEXT,
  
  -- Timing
  created_at TIMESTAMPTZ DEFAULT NOW(),
  started_at TIMESTAMPTZ,
  completed_at TIMESTAMPTZ,
  
  -- Cost
  tokens_used INT DEFAULT 0,
  cost_usd DECIMAL(10,6) DEFAULT 0,
  
  -- Error
  error_message TEXT
);

CREATE INDEX idx_tasks_user ON tasks(user_id);
CREATE INDEX idx_tasks_status ON tasks(status);
```

### scheduled_tasks

For recurring tasks.

```sql
CREATE TABLE public.scheduled_tasks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  
  description TEXT NOT NULL,
  cron_expression TEXT NOT NULL, -- "0 8 * * 1" = Monday 8am
  timezone TEXT DEFAULT 'America/Los_Angeles',
  
  next_run_at TIMESTAMPTZ,
  last_run_at TIMESTAMPTZ,
  is_active BOOLEAN DEFAULT TRUE,
  run_count INT DEFAULT 0,
  
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

## Row Level Security

```sql
-- Enable RLS
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE tasks ENABLE ROW LEVEL SECURITY;
ALTER TABLE scheduled_tasks ENABLE ROW LEVEL SECURITY;

-- Profiles: users see only their own
CREATE POLICY "Users view own profile" ON profiles
  FOR SELECT USING (auth.uid() = id);
CREATE POLICY "Users update own profile" ON profiles
  FOR UPDATE USING (auth.uid() = id);

-- Tasks: users see only their own
CREATE POLICY "Users view own tasks" ON tasks
  FOR SELECT USING (auth.uid() = user_id);

-- Scheduled tasks: users manage their own
CREATE POLICY "Users manage own scheduled" ON scheduled_tasks
  FOR ALL USING (auth.uid() = user_id);
```

## Functions

### Auto-create profile on signup

```sql
CREATE OR REPLACE FUNCTION handle_new_user()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO public.profiles (id, email, username)
  VALUES (
    NEW.id,
    NEW.email,
    LOWER(REGEXP_REPLACE(SPLIT_PART(NEW.email, '@', 1), '[^a-z0-9]', '', 'g'))
  );
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE FUNCTION handle_new_user();
```

### Increment usage

```sql
CREATE OR REPLACE FUNCTION increment_usage(p_user_id UUID)
RETURNS VOID AS $$
BEGIN
  UPDATE profiles
  SET 
    messages_used = messages_used + 1,
    last_active_at = NOW()
  WHERE id = p_user_id;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

## Initial Migration

Run this in Supabase SQL Editor:

```sql
-- 1. Create profiles table
CREATE TABLE public.profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  username TEXT UNIQUE NOT NULL,
  email TEXT NOT NULL,
  display_name TEXT,
  timezone TEXT DEFAULT 'America/Los_Angeles',
  subscription_tier TEXT DEFAULT 'free',
  messages_used INT DEFAULT 0,
  messages_limit INT DEFAULT 20,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  last_active_at TIMESTAMPTZ
);

-- 2. Create tasks table
CREATE TABLE public.tasks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  status TEXT DEFAULT 'pending',
  type TEXT,
  email_subject TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  started_at TIMESTAMPTZ,
  completed_at TIMESTAMPTZ,
  tokens_used INT DEFAULT 0,
  cost_usd DECIMAL(10,6) DEFAULT 0,
  error_message TEXT
);

-- 3. Create scheduled_tasks table
CREATE TABLE public.scheduled_tasks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
  description TEXT NOT NULL,
  cron_expression TEXT NOT NULL,
  timezone TEXT DEFAULT 'America/Los_Angeles',
  next_run_at TIMESTAMPTZ,
  last_run_at TIMESTAMPTZ,
  is_active BOOLEAN DEFAULT TRUE,
  run_count INT DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 4. Indexes
CREATE UNIQUE INDEX idx_profiles_username ON profiles(username);
CREATE INDEX idx_tasks_user ON tasks(user_id);
CREATE INDEX idx_tasks_status ON tasks(status);

-- 5. RLS
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE tasks ENABLE ROW LEVEL SECURITY;
ALTER TABLE scheduled_tasks ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users view own profile" ON profiles FOR SELECT USING (auth.uid() = id);
CREATE POLICY "Users update own profile" ON profiles FOR UPDATE USING (auth.uid() = id);
CREATE POLICY "Users view own tasks" ON tasks FOR SELECT USING (auth.uid() = user_id);
CREATE POLICY "Users manage own scheduled" ON scheduled_tasks FOR ALL USING (auth.uid() = user_id);

-- 6. Trigger for auto-creating profile
CREATE OR REPLACE FUNCTION handle_new_user()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO public.profiles (id, email, username)
  VALUES (
    NEW.id,
    NEW.email,
    LOWER(REGEXP_REPLACE(SPLIT_PART(NEW.email, '@', 1), '[^a-z0-9]', '', 'g'))
  );
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE FUNCTION handle_new_user();

-- 7. Function to increment usage
CREATE OR REPLACE FUNCTION increment_usage(p_user_id UUID)
RETURNS VOID AS $$
BEGIN
  UPDATE profiles
  SET messages_used = messages_used + 1, last_active_at = NOW()
  WHERE id = p_user_id;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```
