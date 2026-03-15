# Nexus 🎯

> Personal productivity app for managing goals, daily tasks, and personal finances — all in one place.

---

## Overview

Nexus is a full-stack web application built with **Next.js 14** that combines goal tracking with personal finance management. Designed for people who want to align their money with their ambitions.

**Core modules:**
- 🎯 **Goals** — Hierarchical goal system with sub-tasks, deadlines, progress tracking, and categories
- 📅 **Daily** — Recurring daily tasks linked to long-term goals
- 💰 **Finance** — Income/expense tracking, monthly budgets, debt & savings tracker
- 📊 **Stats** — Visual dashboard with charts for financial and goal progress

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 14 (App Router) |
| Language | TypeScript |
| Styling | Tailwind CSS |
| UI Components | shadcn/ui |
| Database | Supabase (PostgreSQL) |
| ORM | Prisma |
| Auth | Supabase Auth |
| State Management | Zustand |
| Charts | Recharts |
| Forms | React Hook Form + Zod |
| Deployment | Vercel |

---

## Features

### Goals Module
- Create goals with title, description, category, and deadline
- Add sub-objectives (nested tasks) to each goal
- Progress tracking via % or checklist completion
- Categories: `personal`, `work`, `finance`, `health`, `learning`, `other`
- Mark goals as **daily habits** (recurring)
- Archive / complete goals

### Finance Module
- Log transactions (income / expense) with category, amount, date, and note
- Monthly budget setup per category with overspend alerts
- Debt & savings tracker — track how much you've paid off or saved
- Dashboard with:
  - Monthly balance (income vs expenses)
  - Spending by category (pie chart)
  - Budget progress bars
  - Net worth evolution (line chart)

### Daily View
- Today's pending tasks pulled from active goals
- Habit streak counter
- Quick-complete interface

### Stats Dashboard
- Goal completion rate over time
- Financial health score
- Monthly spending trends
- Savings progress toward targets

---

## Project Structure

```
nexus/
├── app/
│   ├── (auth)/
│   │   ├── login/
│   │   └── register/
│   ├── (dashboard)/
│   │   ├── layout.tsx          ← Sidebar + header shell
│   │   ├── page.tsx            ← Home / daily summary
│   │   ├── goals/
│   │   │   ├── page.tsx        ← Goals list
│   │   │   └── [id]/
│   │   │       └── page.tsx    ← Goal detail + sub-tasks
│   │   ├── finance/
│   │   │   ├── page.tsx        ← Transactions list
│   │   │   ├── budget/
│   │   │   │   └── page.tsx    ← Monthly budgets
│   │   │   └── savings/
│   │   │       └── page.tsx    ← Debts & savings
│   │   └── stats/
│   │       └── page.tsx        ← Analytics dashboard
│   ├── api/
│   │   ├── goals/
│   │   │   └── route.ts
│   │   ├── sub-goals/
│   │   │   └── route.ts
│   │   ├── transactions/
│   │   │   └── route.ts
│   │   ├── budgets/
│   │   │   └── route.ts
│   │   └── savings/
│   │       └── route.ts
│   ├── layout.tsx
│   └── globals.css
│
├── components/
│   ├── goals/
│   │   ├── GoalCard.tsx
│   │   ├── GoalForm.tsx
│   │   ├── SubGoalList.tsx
│   │   └── ProgressBar.tsx
│   ├── finance/
│   │   ├── TransactionItem.tsx
│   │   ├── TransactionForm.tsx
│   │   ├── BudgetCard.tsx
│   │   └── SavingsCard.tsx
│   ├── stats/
│   │   ├── SpendingPieChart.tsx
│   │   ├── MonthlyBarChart.tsx
│   │   └── NetWorthChart.tsx
│   ├── daily/
│   │   └── DailyTaskList.tsx
│   └── ui/                     ← shadcn/ui components
│
├── lib/
│   ├── supabase/
│   │   ├── client.ts           ← Browser Supabase client
│   │   └── server.ts           ← Server Supabase client
│   ├── prisma.ts               ← Prisma client singleton
│   └── utils.ts
│
├── store/
│   ├── useGoalsStore.ts        ← Zustand: goals state
│   ├── useFinanceStore.ts      ← Zustand: finance state
│   └── useUIStore.ts           ← Zustand: sidebar, modals
│
├── types/
│   └── index.ts                ← Shared TypeScript types
│
├── prisma/
│   └── schema.prisma           ← Database schema
│
├── .env.local.example
├── next.config.ts
├── tailwind.config.ts
└── package.json
```

---

## Database Schema

```prisma
model User {
  id           String        @id @default(uuid())
  email        String        @unique
  name         String?
  goals        Goal[]
  transactions Transaction[]
  budgets      Budget[]
  savings      Saving[]
  createdAt    DateTime      @default(now())
}

model Goal {
  id          String      @id @default(uuid())
  userId      String
  user        User        @relation(fields: [userId], references: [id])
  title       String
  description String?
  category    GoalCategory
  deadline    DateTime?
  progress    Int          @default(0)    // 0-100
  isDaily     Boolean      @default(false)
  isCompleted Boolean      @default(false)
  isArchived  Boolean      @default(false)
  subGoals    SubGoal[]
  createdAt   DateTime     @default(now())
  updatedAt   DateTime     @updatedAt
}

model SubGoal {
  id          String   @id @default(uuid())
  goalId      String
  goal        Goal     @relation(fields: [goalId], references: [id], onDelete: Cascade)
  title       String
  isCompleted Boolean  @default(false)
  order       Int      @default(0)
  createdAt   DateTime @default(now())
}

model Transaction {
  id        String          @id @default(uuid())
  userId    String
  user      User            @relation(fields: [userId], references: [id])
  amount    Decimal
  type      TransactionType  // INCOME | EXPENSE
  category  FinanceCategory
  note      String?
  date      DateTime
  createdAt DateTime        @default(now())
}

model Budget {
  id        String          @id @default(uuid())
  userId    String
  user      User            @relation(fields: [userId], references: [id])
  category  FinanceCategory
  limit     Decimal
  month     Int             // 1-12
  year      Int
  createdAt DateTime        @default(now())

  @@unique([userId, category, month, year])
}

model Saving {
  id        String      @id @default(uuid())
  userId    String
  user      User        @relation(fields: [userId], references: [id])
  name      String
  type      SavingType  // SAVING | DEBT
  total     Decimal
  current   Decimal     @default(0)
  note      String?
  createdAt DateTime    @default(now())
  updatedAt DateTime    @updatedAt
}

enum GoalCategory {
  PERSONAL
  WORK
  FINANCE
  HEALTH
  LEARNING
  OTHER
}

enum TransactionType {
  INCOME
  EXPENSE
}

enum FinanceCategory {
  HOUSING
  FOOD
  TRANSPORT
  HEALTH
  ENTERTAINMENT
  SAVINGS
  EDUCATION
  WORK
  OTHER
}

enum SavingType {
  SAVING
  DEBT
}
```

---

## Getting Started

### Prerequisites

- Node.js 18+
- A [Supabase](https://supabase.com) project (free tier works)
- npm / pnpm / yarn

### Installation

```bash
# 1. Clone the repo
git clone https://github.com/AleDev11/nexus.git
cd nexus

# 2. Install dependencies
npm install

# 3. Set up environment variables
cp .env.local.example .env.local
# → Fill in your Supabase URL, anon key, and database URL

# 4. Push the database schema
npx prisma db push

# 5. Run the dev server
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

### Environment Variables

```env
# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://xxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key

# Database (Supabase connection string)
DATABASE_URL=postgresql://postgres:[password]@db.[ref].supabase.co:5432/postgres
DIRECT_URL=postgresql://postgres:[password]@db.[ref].supabase.co:5432/postgres
```

---

## Roadmap

- [x] Project architecture & database schema
- [ ] Auth (login / register with Supabase Auth)
- [ ] Goals CRUD + sub-goals
- [ ] Daily habits view
- [ ] Finance: transactions
- [ ] Finance: budgets
- [ ] Finance: savings & debts
- [ ] Stats dashboard with charts
- [ ] Mobile-responsive UI
- [ ] Dark mode
- [ ] PWA support (installable on mobile)
- [ ] Notifications / reminders

---

## Author

**Alejandro Font** — [afont.dev](https://afont.dev) · [GitHub @AleDev11](https://github.com/AleDev11)