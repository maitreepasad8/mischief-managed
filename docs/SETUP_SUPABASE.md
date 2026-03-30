# Supabase Setup Guide

This guide will walk you through setting up your Supabase project and creating the database tables.

## Step 1: Create a Supabase Project

1. Go to [https://supabase.com/dashboard](https://supabase.com/dashboard)
2. Sign in or create an account
3. Click "New Project"
4. Fill in the details:
   - **Name:** mischief-managed (or your preferred name)
   - **Database Password:** Create a strong password (save this!)
   - **Region:** Choose the closest to you
   - **Pricing Plan:** Free tier is perfect to start
5. Click "Create new project"
6. Wait 2-3 minutes for the project to be provisioned

## Step 2: Get Your API Keys

1. In your Supabase project dashboard, go to **Settings** → **API**
2. You'll see:
   - **Project URL** (e.g., `https://xxxx.supabase.co`)
   - **anon public** key (safe to use in client-side code)
   - **service_role** key (keep this secret! only for server-side)

3. Copy the **Project URL** and **anon public** key

## Step 3: Configure Environment Variables

1. In the project root, create a `.env.local` file:
   ```bash
   cp .env.example .env.local
   ```

2. Edit `.env.local` and paste your keys:
   ```env
   NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
   NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key-here
   ```

3. Save the file. **Never commit `.env.local` to git!** (it's already in .gitignore)

## Step 4: Create Database Tables

We'll use the Supabase SQL Editor to create our tables.

### 4.1: Enable UUID Extension

1. In your Supabase dashboard, go to **SQL Editor**
2. Click **New Query**
3. Paste this SQL:

```sql
-- Enable UUID extension
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```

4. Click **Run** (or press Cmd+Enter)

### 4.2: Create Profiles Table

Paste and run this SQL:

```sql
-- Create profiles table (extends auth.users)
CREATE TABLE IF NOT EXISTS profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  full_name TEXT,
  avatar_url TEXT,
  onboarding_completed BOOLEAN DEFAULT false,
  total_points INTEGER DEFAULT 0,
  preferences JSONB DEFAULT '{
    "notifications_enabled": true,
    "default_reminder_time": "09:00",
    "week_starts_on": "monday",
    "theme": "light"
  }'::jsonb,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Enable RLS
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;

-- RLS Policies for profiles
CREATE POLICY "Users can view own profile"
  ON profiles FOR SELECT
  USING (auth.uid() = id);

CREATE POLICY "Users can update own profile"
  ON profiles FOR UPDATE
  USING (auth.uid() = id);

-- Create trigger to auto-create profile on signup
CREATE OR REPLACE FUNCTION handle_new_user()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO profiles (id, full_name, avatar_url)
  VALUES (
    NEW.id,
    NEW.raw_user_meta_data->>'full_name',
    NEW.raw_user_meta_data->>'avatar_url'
  );
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE FUNCTION handle_new_user();
```

### 4.3: Create Categories Table

```sql
-- Create categories table
CREATE TABLE IF NOT EXISTS categories (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  color TEXT DEFAULT '#FF7A00',
  icon TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(user_id, name)
);

-- Enable RLS
ALTER TABLE categories ENABLE ROW LEVEL SECURITY;

-- RLS Policies
CREATE POLICY "Users can CRUD own categories"
  ON categories FOR ALL
  USING (auth.uid() = user_id);

-- Index
CREATE INDEX idx_categories_user_id ON categories(user_id);
```

### 4.4: Create Tasks Table

```sql
-- Create tasks table
CREATE TABLE IF NOT EXISTS tasks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  title TEXT NOT NULL,
  description TEXT,
  priority TEXT NOT NULL DEFAULT 'medium' CHECK (priority IN ('high', 'medium', 'low')),
  status TEXT NOT NULL DEFAULT 'todo' CHECK (status IN ('todo', 'in_progress', 'done')),
  due_date DATE,
  due_time TIME,
  category TEXT,
  tags TEXT[],
  is_recurring BOOLEAN DEFAULT false,
  recurrence_pattern JSONB,
  completed_at TIMESTAMP WITH TIME ZONE,
  reminder_enabled BOOLEAN DEFAULT false,
  reminder_time TIME,
  position INTEGER DEFAULT 0,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE
);

-- Enable RLS
ALTER TABLE tasks ENABLE ROW LEVEL SECURITY;

-- RLS Policies
CREATE POLICY "Users can CRUD own tasks"
  ON tasks FOR ALL
  USING (auth.uid() = user_id AND deleted_at IS NULL);

-- Indexes
CREATE INDEX idx_tasks_user_id ON tasks(user_id);
CREATE INDEX idx_tasks_due_date ON tasks(due_date);
CREATE INDEX idx_tasks_status ON tasks(status);
CREATE INDEX idx_tasks_user_due_status ON tasks(user_id, due_date, status);
```

### 4.5: Create Habits Table

```sql
-- Create habits table
CREATE TABLE IF NOT EXISTS habits (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  description TEXT,
  icon TEXT,
  color TEXT DEFAULT '#FF7A00',
  category TEXT,
  frequency_type TEXT NOT NULL DEFAULT 'daily' CHECK (frequency_type IN ('daily', 'weekly', 'custom')),
  frequency_days INTEGER[] NOT NULL DEFAULT ARRAY[0,1,2,3,4,5,6],
  goal_type TEXT CHECK (goal_type IN ('count', 'duration', 'boolean')),
  goal_value NUMERIC,
  goal_unit TEXT,
  current_streak INTEGER DEFAULT 0,
  longest_streak INTEGER DEFAULT 0,
  total_completions INTEGER DEFAULT 0,
  reminder_enabled BOOLEAN DEFAULT false,
  reminder_time TIME,
  is_active BOOLEAN DEFAULT true,
  started_at DATE DEFAULT CURRENT_DATE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  deleted_at TIMESTAMP WITH TIME ZONE
);

-- Enable RLS
ALTER TABLE habits ENABLE ROW LEVEL SECURITY;

-- RLS Policies
CREATE POLICY "Users can CRUD own habits"
  ON habits FOR ALL
  USING (auth.uid() = user_id AND deleted_at IS NULL);

-- Indexes
CREATE INDEX idx_habits_user_id ON habits(user_id);
CREATE INDEX idx_habits_is_active ON habits(is_active);
CREATE INDEX idx_habits_user_active ON habits(user_id, is_active);
```

### 4.6: Create Habit Completions Table

```sql
-- Create habit_completions table
CREATE TABLE IF NOT EXISTS habit_completions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  habit_id UUID NOT NULL REFERENCES habits(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  completion_date DATE NOT NULL,
  completed_value NUMERIC,
  notes TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(habit_id, completion_date)
);

-- Enable RLS
ALTER TABLE habit_completions ENABLE ROW LEVEL SECURITY;

-- RLS Policies
CREATE POLICY "Users can CRUD own habit completions"
  ON habit_completions FOR ALL
  USING (auth.uid() = user_id);

-- Indexes
CREATE INDEX idx_habit_completions_habit_id ON habit_completions(habit_id);
CREATE INDEX idx_habit_completions_user_id ON habit_completions(user_id);
CREATE INDEX idx_habit_completions_date ON habit_completions(completion_date);
```

### 4.7: Create Task Completions Table

```sql
-- Create task_completions table (for recurring tasks)
CREATE TABLE IF NOT EXISTS task_completions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  task_id UUID NOT NULL REFERENCES tasks(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  completed_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  completion_date DATE NOT NULL,
  notes TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Enable RLS
ALTER TABLE task_completions ENABLE ROW LEVEL SECURITY;

-- RLS Policies
CREATE POLICY "Users can CRUD own task completions"
  ON task_completions FOR ALL
  USING (auth.uid() = user_id);

-- Indexes
CREATE INDEX idx_task_completions_task_id ON task_completions(task_id);
CREATE INDEX idx_task_completions_user_id ON task_completions(user_id);
CREATE INDEX idx_task_completions_date ON task_completions(completion_date);
```

### 4.8: Create Updated_at Trigger Function

```sql
-- Create function to auto-update updated_at timestamp
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Apply trigger to tables with updated_at column
CREATE TRIGGER update_profiles_updated_at
  BEFORE UPDATE ON profiles
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER update_tasks_updated_at
  BEFORE UPDATE ON tasks
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER update_habits_updated_at
  BEFORE UPDATE ON habits
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
```

## Step 5: Verify Setup

1. In Supabase dashboard, go to **Table Editor**
2. You should see these tables:
   - ✅ profiles
   - ✅ categories
   - ✅ tasks
   - ✅ habits
   - ✅ habit_completions
   - ✅ task_completions

3. Click on each table to verify the columns match the schema

## Step 6: Test Authentication (Optional)

1. Go to **Authentication** → **Users** in Supabase dashboard
2. Click **Add user** → **Create new user**
3. Enter a test email and password
4. Check that a profile was automatically created in the **profiles** table

## Step 7: Start the Development Server

Back in your terminal:

```bash
npm run dev
```

Your app will be running at `http://localhost:3000`

---

## Next Steps

- ✅ Supabase project created
- ✅ Database tables created
- ✅ Environment variables configured
- ✅ Authentication set up

You're now ready to start building the UI and authentication flow!

---

## Troubleshooting

### Error: "relation already exists"
- This means a table was partially created. Drop it and try again:
  ```sql
  DROP TABLE IF EXISTS table_name CASCADE;
  ```

### Error: "permission denied for table"
- RLS might not be enabled. Run:
  ```sql
  ALTER TABLE table_name ENABLE ROW LEVEL SECURITY;
  ```

### Can't authenticate
- Check that your environment variables are correct in `.env.local`
- Restart your dev server after changing environment variables
- Check Supabase logs in the dashboard under **Logs** → **API**
