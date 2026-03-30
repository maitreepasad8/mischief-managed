# Screens & User Flows

## Authentication Flow

### 1. Welcome/Landing Screen
**Purpose:** First screen users see before signing up
- Hero section with app name and tagline
- Key features showcase
- "Sign Up" CTA button (primary)
- "Log In" link (secondary)
- Preview/demo of the app

### 2. Sign Up Screen
**Purpose:** Create new account
- Email input
- Password input
- Confirm password input
- Name input (optional)
- "Create Account" button
- "Already have an account? Log in" link
- Social sign-in options (optional: Google)

### 3. Login Screen
**Purpose:** Existing users sign in
- Email input
- Password input
- "Log In" button
- "Forgot password?" link
- "Don't have an account? Sign up" link

### 4. Onboarding Flow (Optional but recommended)
**Purpose:** First-time user setup
- Welcome message with user's name
- Brief tutorial (swipe through 3-4 cards)
- Set up first habit/task prompt
- Notification permissions request

---

## Main App Screens

### 5. Home/Today Screen ⭐ (Main Screen)
**Purpose:** Daily task overview and quick actions

**Components:**
- **Header**
  - Greeting: "Morning, [Name]" with avatar/emoji
  - Current date

- **Calendar Week View**
  - Current week (Mon-Sun)
  - Current day highlighted
  - Tap to switch days

- **Quick Stats Cards** (3 cards)
  - Tasks Today (🎯) - count of tasks due today
  - Longest Streak (🔥) - best current habit streak (optional for v1)
  - Completed (✅) - total completed tasks/habits all time

- **Reminder/Tip Card** (like "Set the reminder" in reference)
  - Contextual tips
  - Call-to-action button

- **Daily Routine Section**
  - List of today's tasks/habits
  - Each item shows:
    - Checkbox (complete/incomplete)
    - Icon with colored background
    - Task/habit name
    - Streak count (e.g., "Streak 3 days")
    - Time estimate or scheduled time
  - "See all" link

- **Floating Action Button (FAB)**
  - Add new task/habit quickly

**Interactions:**
- Tap checkbox to complete/uncomplete
- Tap item to view details
- Pull to refresh
- Swipe item for quick actions (edit/delete)

### 6. Add/Edit Task Screen
**Purpose:** Create or modify a task

**Form Fields:**
- Task name (text input)
- Description (optional, textarea)
- Priority (High/Medium/Low selector)
- Due date (date picker)
- Due time (time picker, optional)
- Category/Tags (optional dropdown)
- Repeat settings (if recurring)
- Reminders toggle

**Actions:**
- "Save Task" button
- Cancel/Back button
- Delete button (if editing)

### 7. Add/Edit Habit Screen (like reference "New habit")
**Purpose:** Create or modify a habit

**Form Fields:**
- Habit name with illustration/emoji
- Goal toggle (optional numeric goal)
  - Add date (target completion date)
  - Add amount (e.g., "5 times per week")
- Repeat days selector (M T W T F S S buttons)
- Reminder toggle
- Category/icon selector

**Actions:**
- "Save Habit" button (prominent orange)
- Cancel/Back button

### 8. All Tasks Screen
**Purpose:** View all tasks (not just today's)

**Filters/Views:**
- Tabs: All / Active / Completed
- Sort options: Date / Priority / Category
- Search bar

**Task List:**
- Grouped by status or date
- Same task card design as Home screen
- Infinite scroll or pagination

### 9. Habits Screen
**Purpose:** View all habits and their streaks

**Layout:**
- List of all habits
- Each habit card shows:
  - Icon with background
  - Habit name
  - Current streak
  - Completion status for today
  - Weekly progress indicator (mini calendar)

**Actions:**
- Tap to view habit details
- Check in for today
- Add new habit (FAB)

### 10. Habit Detail/History Screen
**Purpose:** See full history and stats for a habit

**Content:**
- Habit name and icon
- Current streak (large display)
- Total completions
- Calendar heatmap (like GitHub contributions)
- Weekly/Monthly completion rate
- Best streak record
- Notes section (optional)

**Actions:**
- Edit habit
- Delete habit
- Mark as complete for today

### 11. Progress & Insights Screen (like reference)
**Purpose:** Visualizations and analytics

**Charts/Stats:**
- **Bar Chart:** Habit completion percentages by category
  - Walking, Running, Meditation, Drink water, etc.
  - Percentage labels on bars

- **Points/Gamification Section**
  - Total points earned this week
  - Breakdown: Food / Forget area / Time
  - Actual values (440 lb, 200 ft², 7h 30m)

- **Share Progress** button

**Time Filters:**
- This Week / This Month / All Time

### 12. Profile/Settings Screen
**Purpose:** User account and app settings

**Sections:**
- **Profile**
  - Avatar/photo
  - Name
  - Email
  - Change password

- **Preferences**
  - Notifications toggle
  - Reminder times
  - Theme (light/dark - future)
  - Start of week (Mon/Sun)

- **Data**
  - Export data
  - Import data
  - Clear completed tasks

- **About**
  - App version
  - Privacy policy
  - Terms of service

- **Sign Out** button

---

## Additional Screens (Future Enhancements)

### 13. Calendar View (Full Month)
**Purpose:** Month-at-a-glance view of tasks/habits

### 14. Categories/Tags Management
**Purpose:** Create and organize custom categories

### 15. Achievements/Badges Screen
**Purpose:** Gamification - show earned badges and milestones

### 16. Social/Sharing Screen
**Purpose:** Share progress with friends or accountability partners

---

## Navigation Structure

**Primary Navigation (Bottom Tab Bar):**
1. 🏠 Home (Today screen)
2. ✅ Tasks (All tasks screen)
3. 🔥 Habits (Habits screen)
4. 📊 Progress (Progress & insights screen)
5. 👤 Profile (Settings screen)

**Secondary Navigation:**
- Top app bar with back button and screen title
- Floating Action Button (FAB) for quick add
- Modal sheets for add/edit forms

---

## User Flows

### Flow 1: Complete a Task
1. User opens Home screen
2. User taps checkbox on task
3. Task animates to completed state (strikethrough, fade)
4. Celebration micro-animation (confetti/checkmark)
5. Stats update (completed count increases)

### Flow 2: Add a New Habit
1. User taps FAB on Home or Habits screen
2. "Add Task or Habit?" modal appears
3. User selects "Habit"
4. New Habit screen opens
5. User fills in habit details
6. User taps "Save Habit"
7. Habit appears in list with animation
8. Success message appears

### Flow 3: View Progress
1. User navigates to Progress tab
2. Charts load with animation
3. User scrolls to see different metrics
4. User taps time filter (Week/Month/All)
5. Charts update with filtered data

### Flow 4: First Time User
1. User signs up
2. Onboarding screens (swipe through)
3. Prompt to add first habit
4. Add Habit screen opens
5. User creates first habit
6. Redirect to Home screen
7. Welcome message/tutorial highlights

---

## Screen Priority for MVP

**Phase 1 (Core):**
1. Sign Up
2. Login
3. Home/Today Screen
4. Add/Edit Task Screen
5. Add/Edit Habit Screen

**Phase 2 (Essential):**
6. All Tasks Screen
7. Habits Screen
8. Profile/Settings Screen

**Phase 3 (Polish):**
9. Progress & Insights Screen
10. Habit Detail Screen
11. Onboarding Flow

---

## Design Notes

**Color Palette (from reference):**
- Background: Soft cream/beige (#F5F3F0)
- Primary CTA: Vibrant orange (#FF7A00)
- Success/Complete: Green (#6FB865)
- Accent colors: Soft peach, pink, brown, purple
- Text: Dark brown/black (#2D2D2D)
- Secondary text: Gray (#8A8A8A)

**Design Principles:**
- Generous rounded corners (16-24px)
- Soft shadows for depth
- Playful icons with circular backgrounds
- Warm, organic color palette
- Mobile-first responsive design
- Smooth animations and transitions
- Encouraging, friendly copy
