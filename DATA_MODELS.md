# Data Models & Database Schema

## Overview
This document defines the database schema for Supabase (PostgreSQL). All tables will use Supabase's built-in authentication and Row Level Security (RLS) policies.

---

## Core Tables

### 1. users (managed by Supabase Auth)
**Purpose:** User authentication and profile data

Supabase provides this table automatically through `auth.users`. We'll extend it with a custom `profiles` table.

**Fields (Supabase Auth):**
- `id` (uuid, primary key) - auto-generated
- `email` (text, unique)
- `encrypted_password` (text)
- `created_at` (timestamp)
- `updated_at` (timestamp)
- `email_confirmed_at` (timestamp)

### 2. profiles
**Purpose:** Extended user profile information

**Fields:**
- `id` (uuid, primary key, references auth.users.id)
- `full_name` (text, nullable)
- `avatar_url` (text, nullable)
- `onboarding_completed` (boolean, default: false)
- `total_points` (integer, default: 0) // for gamification (future)
- `preferences` (jsonb, nullable)
  ```json
  {
    "notifications_enabled": true,
    "default_reminder_time": "09:00",
    "week_starts_on": "monday",
    "theme": "light"
  }
  ```
- `created_at` (timestamp, default: now())
- `updated_at` (timestamp, default: now())

**Indexes:**
- Primary key on `id`

**RLS Policies:**
- Users can read their own profile
- Users can update their own profile
- New profiles are created on signup (trigger)

---

### 3. tasks
**Purpose:** One-time or recurring tasks

**Fields:**
- `id` (uuid, primary key, default: gen_random_uuid())
- `user_id` (uuid, references profiles.id, not null)
- `title` (text, not null)
- `description` (text, nullable)
- `priority` (text, not null)
  - Values: 'high', 'medium', 'low'
  - Default: 'medium'
- `status` (text, not null)
  - Values: 'todo', 'in_progress', 'done'
  - Default: 'todo'
- `due_date` (date, nullable)
- `due_time` (time, nullable)
- `category` (text, nullable)
  - Examples: 'work', 'personal', 'health', 'learning'
- `tags` (text[], nullable)
- `is_recurring` (boolean, default: false)
- `recurrence_pattern` (jsonb, nullable)
  ```json
  {
    "frequency": "daily" | "weekly" | "monthly",
    "interval": 1,
    "days_of_week": [1, 3, 5], // Mon, Wed, Fri (for weekly)
    "end_date": "2025-12-31"
  }
  ```
- `completed_at` (timestamp, nullable)
- `reminder_enabled` (boolean, default: false)
- `reminder_time` (time, nullable)
- `position` (integer, default: 0) // for custom ordering
- `created_at` (timestamp, default: now())
- `updated_at` (timestamp, default: now())
- `deleted_at` (timestamp, nullable) // soft delete

**Indexes:**
- Primary key on `id`
- Index on `user_id`
- Index on `due_date`
- Index on `status`
- Composite index on `(user_id, due_date, status)`

**RLS Policies:**
- Users can CRUD their own tasks only
- Filter by `user_id = auth.uid()`

---

### 4. habits
**Purpose:** Recurring habits to track

**Fields:**
- `id` (uuid, primary key, default: gen_random_uuid())
- `user_id` (uuid, references profiles.id, not null)
- `name` (text, not null)
- `description` (text, nullable)
- `icon` (text, nullable) // emoji or icon identifier
- `color` (text, default: '#FF7A00') // hex color for icon background
- `category` (text, nullable)
  - Examples: 'health', 'fitness', 'mindfulness', 'productivity'
- `frequency_type` (text, not null)
  - Values: 'daily', 'weekly', 'custom'
  - Default: 'daily'
- `frequency_days` (integer[], not null)
  - Array of day numbers: 0=Sunday, 1=Monday, ..., 6=Saturday
  - Example: [1,2,3,4,5] for weekdays
  - For daily: [0,1,2,3,4,5,6]
- `goal_type` (text, nullable)
  - Values: 'count', 'duration', 'boolean'
  - 'boolean': just check yes/no
  - 'count': track number (e.g., 5 glasses of water)
  - 'duration': track time (e.g., 30 minutes meditation)
- `goal_value` (numeric, nullable)
  - Target value for count/duration goals
- `goal_unit` (text, nullable)
  - Examples: 'minutes', 'glasses', 'pages', 'reps'
- `current_streak` (integer, default: 0)
- `longest_streak` (integer, default: 0)
- `total_completions` (integer, default: 0)
- `reminder_enabled` (boolean, default: false)
- `reminder_time` (time, nullable)
- `is_active` (boolean, default: true)
- `started_at` (date, default: today())
- `created_at` (timestamp, default: now())
- `updated_at` (timestamp, default: now())
- `deleted_at` (timestamp, nullable) // soft delete

**Indexes:**
- Primary key on `id`
- Index on `user_id`
- Index on `is_active`
- Composite index on `(user_id, is_active)`

**RLS Policies:**
- Users can CRUD their own habits only

---

### 5. habit_completions
**Purpose:** Track when habits are completed (for history/heatmap)

**Fields:**
- `id` (uuid, primary key, default: gen_random_uuid())
- `habit_id` (uuid, references habits.id, not null, on delete: cascade)
- `user_id` (uuid, references profiles.id, not null)
- `completion_date` (date, not null)
- `completed_value` (numeric, nullable)
  - For count/duration goals: actual value achieved
  - For boolean goals: null or 1
- `notes` (text, nullable)
- `created_at` (timestamp, default: now())

**Unique Constraint:**
- `(habit_id, completion_date)` - one completion per day per habit

**Indexes:**
- Primary key on `id`
- Unique index on `(habit_id, completion_date)`
- Index on `user_id`
- Index on `completion_date`

**RLS Policies:**
- Users can CRUD their own completions only

---

### 6. task_completions
**Purpose:** Track completion history for recurring tasks

**Fields:**
- `id` (uuid, primary key, default: gen_random_uuid())
- `task_id` (uuid, references tasks.id, not null, on delete: cascade)
- `user_id` (uuid, references profiles.id, not null)
- `completed_at` (timestamp, not null, default: now())
- `completion_date` (date, not null) // extracted from completed_at for queries
- `notes` (text, nullable)

**Indexes:**
- Primary key on `id`
- Index on `task_id`
- Index on `user_id`
- Index on `completion_date`

**RLS Policies:**
- Users can CRUD their own completions only

---

### 7. categories
**Purpose:** User-defined categories for tasks/habits

**Fields:**
- `id` (uuid, primary key, default: gen_random_uuid())
- `user_id` (uuid, references profiles.id, not null)
- `name` (text, not null)
- `color` (text, default: '#FF7A00')
- `icon` (text, nullable)
- `created_at` (timestamp, default: now())

**Unique Constraint:**
- `(user_id, name)` - no duplicate category names per user

**Indexes:**
- Primary key on `id`
- Unique index on `(user_id, name)`

**RLS Policies:**
- Users can CRUD their own categories only

---

### 8. achievements (Future Enhancement)
**Purpose:** Gamification badges and milestones

**Fields:**
- `id` (uuid, primary key)
- `user_id` (uuid, references profiles.id, not null)
- `achievement_type` (text, not null)
  - Examples: 'first_habit', '7_day_streak', '30_day_streak', '100_tasks_completed'
- `earned_at` (timestamp, default: now())
- `metadata` (jsonb, nullable)

**Indexes:**
- Primary key on `id`
- Index on `user_id`

**RLS Policies:**
- Users can read their own achievements
- System/backend creates achievements (no direct user insert)

---

## Database Functions & Triggers

### 1. handle_new_user()
**Purpose:** Create profile when new user signs up

```sql
CREATE FUNCTION handle_new_user()
RETURNS trigger AS $$
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

### 2. update_updated_at()
**Purpose:** Auto-update updated_at timestamp

```sql
CREATE FUNCTION update_updated_at()
RETURNS trigger AS $$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Apply to all tables with updated_at
CREATE TRIGGER update_profiles_updated_at
  BEFORE UPDATE ON profiles
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();

-- Repeat for tasks, habits, etc.
```

### 3. calculate_habit_streak() (Optional - for real-time streak calculation)
**Purpose:** Calculate current streak for a habit on-demand

**Note:** Streaks are stored in the `habits` table and updated via triggers when completions are added. This function is for validation or recalculation if needed.

```sql
CREATE FUNCTION calculate_habit_streak(habit_uuid uuid)
RETURNS integer AS $$
DECLARE
  streak_count integer := 0;
  check_date date := CURRENT_DATE;
  habit_days integer[];
BEGIN
  -- Get the habit's frequency days
  SELECT frequency_days INTO habit_days
  FROM habits WHERE id = habit_uuid;

  -- Loop backwards from today, only checking scheduled days
  LOOP
    -- Check if this day is a scheduled habit day
    IF EXTRACT(DOW FROM check_date)::integer = ANY(habit_days) THEN
      IF EXISTS (
        SELECT 1 FROM habit_completions
        WHERE habit_id = habit_uuid
        AND completion_date = check_date
      ) THEN
        streak_count := streak_count + 1;
        check_date := check_date - INTERVAL '1 day';
      ELSE
        EXIT; -- Streak broken
      END IF;
    ELSE
      -- Skip non-scheduled days
      check_date := check_date - INTERVAL '1 day';
    END IF;
  END LOOP;

  RETURN streak_count;
END;
$$ LANGUAGE plpgsql;
```

---

## Row Level Security (RLS) Policies

### Enable RLS on all tables:
```sql
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE tasks ENABLE ROW LEVEL SECURITY;
ALTER TABLE habits ENABLE ROW LEVEL SECURITY;
ALTER TABLE habit_completions ENABLE ROW LEVEL SECURITY;
ALTER TABLE task_completions ENABLE ROW LEVEL SECURITY;
ALTER TABLE categories ENABLE ROW LEVEL SECURITY;
```

### Example Policy (tasks):
```sql
-- Users can only see their own tasks
CREATE POLICY "Users can view own tasks"
  ON tasks FOR SELECT
  USING (auth.uid() = user_id);

-- Users can only insert their own tasks
CREATE POLICY "Users can insert own tasks"
  ON tasks FOR INSERT
  WITH CHECK (auth.uid() = user_id);

-- Users can only update their own tasks
CREATE POLICY "Users can update own tasks"
  ON tasks FOR UPDATE
  USING (auth.uid() = user_id);

-- Users can only delete their own tasks
CREATE POLICY "Users can delete own tasks"
  ON tasks FOR DELETE
  USING (auth.uid() = user_id);
```

---

## Relationships

```
users (auth.users)
  ↓
profiles (1:1)
  ↓
  ├─ tasks (1:many)
  │    ↓
  │    └─ task_completions (1:many)
  │
  ├─ habits (1:many)
  │    ↓
  │    └─ habit_completions (1:many)
  │
  ├─ categories (1:many)
  │
  └─ achievements (1:many)
```

---

## Sample Data

### Sample Task:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "user_id": "123e4567-e89b-12d3-a456-426614174000",
  "title": "Design task tracker UI",
  "description": "Create mockups in Figma",
  "priority": "high",
  "status": "in_progress",
  "due_date": "2025-03-30",
  "due_time": "14:00:00",
  "category": "work",
  "tags": ["design", "figma"],
  "is_recurring": false,
  "reminder_enabled": true,
  "reminder_time": "13:00:00"
}
```

### Sample Habit:
```json
{
  "id": "660e8400-e29b-41d4-a716-446655440001",
  "user_id": "123e4567-e89b-12d3-a456-426614174000",
  "name": "Morning Meditation",
  "icon": "🧘",
  "color": "#B5A8D6",
  "category": "mindfulness",
  "frequency_type": "daily",
  "frequency_days": [0, 1, 2, 3, 4, 5, 6],
  "goal_type": "duration",
  "goal_value": 15,
  "goal_unit": "minutes",
  "current_streak": 12,
  "longest_streak": 28,
  "total_completions": 156,
  "reminder_enabled": true,
  "reminder_time": "07:00:00"
}
```

### Sample Habit Completion:
```json
{
  "id": "770e8400-e29b-41d4-a716-446655440002",
  "habit_id": "660e8400-e29b-41d4-a716-446655440001",
  "user_id": "123e4567-e89b-12d3-a456-426614174000",
  "completion_date": "2025-03-30",
  "completed_value": 15,
  "notes": "Felt really centered today"
}
```

---

## Migration Order

1. Enable UUID extension
2. Create profiles table + trigger
3. Create categories table
4. Create tasks table
5. Create task_completions table
6. Create habits table
7. Create habit_completions table
8. Create achievements table (future)
9. Enable RLS on all tables
10. Create RLS policies
11. Create helper functions (streaks, etc.)

---

## API Considerations

### Computed Fields (via Supabase views or functions):
- `today_tasks` - tasks due today
- `overdue_tasks` - tasks past due date
- `active_habits_today` - habits scheduled for today
- `completed_habits_today` - habits completed today
- `weekly_completion_rate` - percentage of habits completed this week

### Indexes for Performance:
- Composite indexes on frequently queried columns
- Partial indexes for common filters (e.g., active habits, incomplete tasks)
