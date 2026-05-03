# 🎨 VISUAL INTERVIEW REFERENCE
## Diagrams, Flowcharts & Visual Concepts

---

## 🏗️ ARCHITECTURE DIAGRAM

```
┌─────────────────────────────────────────────────────────────────┐
│                        VERCEL (DEPLOYMENT)                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌────────────────────┐         ┌──────────────────────┐        │
│  │  FRONTEND LAYER    │         │   API LAYER          │        │
│  │  (Next.js Pages)   │◄───────►│  (API Routes)        │        │
│  ├────────────────────┤         ├──────────────────────┤        │
│  │ • React Components │         │ • GET Handlers       │        │
│  │ • "use client"     │         │ • POST Handlers      │        │
│  │ • Forms            │         │ • PUT Handlers       │        │
│  │ • State (auth)     │         │ • PATCH Handlers     │        │
│  │ • Navigation       │         │ • DELETE Handlers    │        │
│  └────────────────────┘         │ • Status Codes       │        │
│                                  │ • Validation         │        │
│                                  │ • Auth Checks        │        │
│                                  └──────────┬───────────┘        │
│                                             │                     │
│                   ┌─────────────────────────┘                    │
│                   │                                               │
│                   ▼                                               │
│  ┌──────────────────────────────┐                               │
│  │   DATABASE LAYER             │                               │
│  │   (MongoDB)                  │                               │
│  ├──────────────────────────────┤                               │
│  │ • Users                      │                               │
│  │ • Donors                     │                               │
│  │ • Payments                   │                               │
│  │ • Expenditures               │                               │
│  │ • Properties                 │                               │
│  │ • Settings                   │                               │
│  │ • AuditLogs                  │                               │
│  └──────────────────────────────┘                               │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

**The Flow:**
```
User in Browser
        ↓
React Component (Frontend)
        ↓
API Request (HTTP GET/POST/etc)
        ↓
Next.js API Route Handler
        ↓
Validate Input → Check Auth → Query MongoDB → Return Response
        ↓
HTTP Status Code + JSON Response
        ↓
Browser → React Component Updates
```

---

## 📡 REQUEST/RESPONSE CYCLE

```
REQUEST ARRIVES
        │
        ├─→ Is input valid?
        │   NO: Return 400 Bad Request
        │
        ├─→ Is user authenticated?
        │   NO: Return 401 Unauthorized
        │
        ├─→ Does user have permission?
        │   NO: Return 403 Forbidden
        │
        ├─→ Connect to MongoDB ✓
        │
        ├─→ Does resource exist?
        │   NO: Return 404 Not Found
        │
        ├─→ Execute database operation ✓
        │
        ├─→ Return success with data ✓
        │   (200 OK or 201 Created)
        │
        └─→ Catch any error → 500 Internal Server Error
```

---

## 🔐 AUTHENTICATION FLOW

```
┌─────────────────────────────────────────────────────────────┐
│                  USER LOGIN PROCESS                         │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  Browser                    API Route                        │
│     │                           │                           │
│     │─ POST /api/auth/login ───►│                           │
│     │  (username, password)     │                           │
│     │                      ┌────┴─────────────────┐         │
│     │                      │ 1. Validate input    │         │
│     │                      │ 2. Query MongoDB     │         │
│     │                      │ 3. Compare password  │         │
│     │                      │ 4. Return user+role  │         │
│     │                      └────┬─────────────────┘         │
│     │◄─ {user, role} (200) ─────│                           │
│     │                           │                           │
│  ┌──┴──────────────┐                                       │
│  │ Store user data │                                       │
│  │ Store role      │                                       │
│  │ Save in context │                                       │
│  └──────────────────┘                                      │
│                                                              │
├─────────────────────────────────────────────────────────────┤
│              SUBSEQUENT REQUESTS (PROTECTED)                │
│                                                              │
│  Browser                    API Route                       │
│     │                           │                          │
│     │─ GET /api/users ──────────►│                          │
│     │  x-user-role: Admin        │                          │
│     │  (in headers)         ┌────┴──────────────┐           │
│     │                       │ 1. Check header   │           │
│     │                       │ 2. Is Admin? ✓    │           │
│     │                       │ 3. Query users    │           │
│     │                       │ 4. Return users   │           │
│     │                       └────┬──────────────┘           │
│     │◄─ {users} (200) ──────────│                           │
│     │                           │                           │
│  vs Non-Admin:                                             │
│     │                           │                          │
│     │─ GET /api/users ──────────►│                          │
│     │  x-user-role: Viewer       │                          │
│     │                      ┌─────┴──────────────┐           │
│     │                      │ 1. Check header    │           │
│     │                      │ 2. Is Admin? NO ✗  │           │
│     │                      │ 3. Return 403      │           │
│     │                      └─────┬──────────────┘           │
│     │◄─ {error} (403) ──────────│                           │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 🎯 HTTP STATUS CODE DECISION TREE

```
                        REQUEST RECEIVED
                              │
                ┌─────────────┼──────────────┐
                │             │              │
            Is it a     Has input?      Error in code?
            POST?          │                 │
             │             │                 │
         YES│ NO           │                 │
         ┌───┴──┐      BAD │ GOOD        YES│
         │      │      ┌───┴───┐            │
         │      │      │       │         500
        201    200    400    Check         Internal
       Created  OK   Bad     Auth          Error
             Request  │
                      │
           ┌──────────┴──────────┐
           │                     │
         INVALID              VALID
           │                     │
        401                   Check
       Wrong                Permission
       Password               │
                    ┌─────────┴─────────┐
                    │                   │
                   YES                  NO
                    │                   │
                   OK → GET            403
                  Resource?          Forbidden
                    │
         ┌──────────┴──────────┐
         │                     │
       YES                    NO
         │                     │
       200                   404
       OK                 Not Found
```

---

## 📋 YOUR API ENDPOINTS VISUALIZATION

```
┌─ /api/auth
│  └─ /login                    [POST] → 200/401
│  └─ /verify-password          [POST] → 200/404/401
│
├─ /api/users
│  ├─ [ROOT]                    [GET]  → 200/403 (admin only)
│  ├─ [ROOT]                    [POST] → 201/400/403 (admin only)
│  └─ /[id]                     [PATCH] → 200/400/403
│
├─ /api/donors
│  ├─ [ROOT]                    [GET]  → 200
│  ├─ [ROOT]                    [POST] → 201/400
│  └─ /[id]
│     ├─                         [GET]  → 200/404
│     ├─                         [PUT]  → 200/400/404
│     └─                         [DELETE] → 200/404
│
├─ /api/payments
│  ├─ [ROOT]                    [GET]  → 200
│  ├─ [ROOT]                    [POST] → 201/400
│  └─ /[id]                     [DELETE] → 200/404
│
├─ /api/expenditures
│  ├─ [ROOT]                    [GET]  → 200
│  ├─ [ROOT]                    [POST] → 201/400
│  └─ /[id]
│     ├─                         [PUT]  → 200/400/404
│     └─                         [DELETE] → 200/404
│
├─ /api/properties
│  ├─ [ROOT]                    [GET]  → 200
│  ├─ [ROOT]                    [POST] → 201/400
│  └─ /[id]
│     └─                         [PUT]  → 200/400/404
│
├─ /api/settings
│  ├─ [ROOT]                    [GET]  → 200
│  └─ [ROOT]                    [PUT]  → 200/400/403 (admin)
│
├─ /api/dashboard
│  └─ /stats                    [GET]  → 200 (with filters)
│
└─ /api/reports
   └─ /ledger                   [GET]  → 200 (financial report)

Legend: [ROOT] = no id in path, [GET] = HTTP method, 200 = status code
```

---

## ⚙️ API HANDLER ANATOMY

```
┌────────────────────────────────────────────────────────────────┐
│                  API ROUTE HANDLER STRUCTURE                  │
├────────────────────────────────────────────────────────────────┤
│                                                                  │
│  export async function POST(req: NextRequest) {                │
│    try {                                                         │
│                                                                  │
│      1️⃣  SETUP                                                  │
│      └─ await connectDB()          ← Connect to DB             │
│                                                                  │
│      2️⃣  EXTRACT DATA                                           │
│      ├─ const body = await req.json()                          │
│      ├─ const params = await params (for [id])                 │
│      └─ const headers = req.headers.get("x-header")            │
│                                                                  │
│      3️⃣  VALIDATE INPUT                                        │
│      ├─ if (!username) return json({error}, 400)              │
│      ├─ if (role not in allowed) return json({error}, 400)    │
│      └─ if (duplicate) return json({error}, 400)              │
│                                                                  │
│      4️⃣  CHECK AUTHORIZATION                                   │
│      └─ if (role !== "Admin") return json({error}, 403)       │
│                                                                  │
│      5️⃣  DATABASE OPERATION                                    │
│      ├─ const user = await User.create({...})                 │
│      └─ if (!user) return json({error}, 404)                  │
│                                                                  │
│      6️⃣  RETURN SUCCESS                                        │
│      └─ return json({user}, {status: 201})                    │
│                                                                  │
│    } catch (error) {                                            │
│      return json({error: "message"}, {status: 500})            │
│    }                                                            │
│  }                                                               │
│                                                                  │
└────────────────────────────────────────────────────────────────┘
```

---

## 🔑 STATUS CODE LEGEND

```
┌──────────────────────────────────────────────────────────────────┐
│  2XX SUCCESS                                                      │
├──────────────────────────────────────────────────────────────────┤
│  200 OK                  ✓ Success (GET, PUT, PATCH)             │
│  201 Created             ✓ Resource created (POST)               │
│                                                                   │
├──────────────────────────────────────────────────────────────────┤
│  4XX CLIENT ERROR                                                 │
├──────────────────────────────────────────────────────────────────┤
│  400 Bad Request         ✗ Validation failed (invalid input)     │
│  401 Unauthorized        ✗ Authentication failed (no password)   │
│  403 Forbidden           ✗ Authorization failed (no permission)  │
│  404 Not Found           ✗ Resource doesn't exist                │
│                                                                   │
├──────────────────────────────────────────────────────────────────┤
│  5XX SERVER ERROR                                                 │
├──────────────────────────────────────────────────────────────────┤
│  500 Internal Server Err ✗ Unhandled exception / DB error        │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘

MEMORY TRICK:
  2XX: Happy (everything worked)
  4XX: Your fault (bad request, auth issue, not found)
  5XX: Our fault (server broke)
```

---

## 🗄️ DATABASE RELATIONSHIPS

```
┌──────────────────────────────────────────────────────────────┐
│                   YOUR DATA MODEL                            │
├──────────────────────────────────────────────────────────────┤
│                                                                │
│  ┌─────────┐                          ┌──────────┐           │
│  │  Users  │                          │ Settings │           │
│  ├─────────┤                          ├──────────┤           │
│  │ id      │────┐                     │ id       │           │
│  │ username│    │                     │ appName  │           │
│  │ password│    │                     │ timezone │           │
│  │ email   │    │                     └──────────┘           │
│  │ role    │    │                                             │
│  │ status  │    │      ┌────────────────┐                    │
│  └─────────┘    │      │   Audit Log    │                    │
│                 │      ├────────────────┤                    │
│                 └─────►│ performedBy    │                    │
│                        │ action         │                    │
│                        │ model          │                    │
│                        │ recordId       │                    │
│                        └────────────────┘                    │
│                                                                │
│  ┌─────────┐                          ┌──────────────┐      │
│  │ Donors  │────┐          ┌─────────►│  Properties  │      │
│  ├─────────┤    │          │          ├──────────────┤      │
│  │ id      │    │          │          │ id           │      │
│  │ name    │    │          │          │ address      │      │
│  │ email   │    │          │          │ value        │      │
│  │ phone   │    │          │          │ type         │      │
│  │ status  │    │          │          └──────────────┘      │
│  └─────────┘    │          │                                 │
│       │         │          │                                 │
│       └────────►│          │                                 │
│                 │          │                                 │
│                 ▼          │                                 │
│         ┌──────────────┐   │                                 │
│         │  Payments    │───┘                                 │
│         ├──────────────┤                                     │
│         │ donor_id ◄───┘                                     │
│         │ amount       │                                     │
│         │ method       │                                     │
│         │ date         │                                     │
│         │ month/year   │                                     │
│         └──────────────┘                                     │
│                                                                │
│         ┌──────────────┐                                     │
│         │ Expenditures │                                     │
│         ├──────────────┤                                     │
│         │ category     │                                     │
│         │ amount       │                                     │
│         │ date         │                                     │
│         │ description  │                                     │
│         └──────────────┘                                     │
│                                                                │
└──────────────────────────────────────────────────────────────┘

Key Relationships:
  Users ──creates──► Payments, Expenditures (audit)
  Donors ◄──has────── Payments
  Donors ──own────────► Properties
```

---

## 🔄 NEXT.JS APP ROUTER NAVIGATION

```
YOUR PROJECT DIRECTORY STRUCTURE:

/app (App Router)
│
├── page.tsx                     ← "/" route (Home → redirects to /sign-in)
├── layout.tsx                   ← Root layout
│
├── /sign-in                     ← "/sign-in" route
│   └── page.tsx
│   └── [[...sign-in]]/          ← Catch-all for Clerk UI
│
├── /dashboard                   ← "/dashboard" route
│   ├── layout.tsx               ← Dashboard layout
│   ├── page.tsx                 ← Dashboard home
│   ├── /beneficiaries/
│   ├── /donors/
│   ├── /expenditures/
│   ├── /payments/
│   ├── /properties/
│   ├── /reports/
│   ├── /settings/
│   └── /users/
│
└── /api (API Routes)
    ├── /auth
    │   ├── /login/route.ts      ← POST /api/auth/login
    │   └── /verify-password/route.ts
    │
    ├── /users/route.ts          ← GET/POST /api/users
    ├── /users/[id]/route.ts     ← PATCH /api/users/[id]
    │
    ├── /donors/route.ts         ← GET/POST /api/donors
    ├── /donors/[id]/route.ts    ← GET/PUT/DELETE
    │
    ├── /payments/route.ts
    ├── /payments/[id]/route.ts
    │
    ├── /expenditures/route.ts
    ├── /expenditures/[id]/route.ts
    │
    ├── /properties/route.ts
    ├── /properties/[id]/route.ts
    │
    ├── /settings/route.ts
    │
    ├── /dashboard/
    │   └── /stats/route.ts      ← GET /api/dashboard/stats
    │
    └── /reports/
        └── /ledger/route.ts     ← GET /api/reports/ledger

Route Examples:
  GET /                          → app/page.tsx (redirects)
  GET /sign-in                   → app/sign-in/page.tsx
  GET /dashboard                 → app/dashboard/page.tsx
  GET /dashboard/donors          → app/dashboard/donors/page.tsx
  POST /api/auth/login           → app/api/auth/login/route.ts
  GET /api/donors/123            → app/api/donors/[id]/route.ts (GET handler)
  PUT /api/donors/123            → app/api/donors/[id]/route.ts (PUT handler)
```

---

## 🎓 LEARNING PYRAMID

```
                            ▲
                           ╱ ╲     MASTERY
                          ╱   ╲    ─────
                         ╱ Can ╲   Build production
                        ╱ Teach ╲  features from
                       ╱   &    ╲  scratch
                      ╱  Mentor  ╲
                     ╱────────────╲
                    ╱              ╲   EXPERT
                   ╱ Can Debug &   ╲  ─────
                  ╱  Explain Why   ╲  Fix bugs, optimize
                 ╱────────────────────╲
                ╱                      ╲  COMPETENT
               ╱   Can Build With      ╲ ────────
              ╱      Reference         ╲ Follow patterns,
             ╱──────────────────────────╲ copy code
            ╱                            ╲
           ╱      Can Understand         ╱ FAMILIAR
          ╱──────────────────────────────╱ ────────
         ╱                              ╱  Read & explain
        ╱         Concepts              ╱   your code
       ╱──────────────────────────────╱
      ╱            Know              ╱    BEGINNER
     ╱   (Memorized facts)           ╱    ────────
    ╱──────────────────────────────╱      HTTP status
   ╱                              ╱        codes list
  ╱──────────────────────────────╱
        YOU ARE HERE

Goal: Reach EXPERT level for interview
→ You can explain AND debug your code
→ You understand the why, not just the how
```

---

## 📊 INTERVIEW SUCCESS FORMULA

```
                    YOU'RE HIRED! 🎉
                         ↑
                         │
        ┌────────────────┴────────────────┐
        │                                 │
    Technical Knowledge      Communication
         75%                      25%
         │                         │
         ├─ HTTP status codes     ├─ Clear explanations
         ├─ API route patterns    ├─ Real examples from code
         ├─ Auth flow            ├─ Admit what you don't know
         ├─ Database operations  ├─ Ask good questions
         ├─ Error handling       └─ Positive attitude
         └─ Your project code


YOU HAVE:
  ✅ Technical Knowledge (from these guides)
  ✅ Real Project (to reference)
  ✅ Code Examples (to explain)
  
PRACTICE:
  🎯 Explanations (speak clearly)
  🎯 Diagrams (draw on whiteboard)
  🎯 Admitting gaps (shows maturity)
```

---

## 🎤 THE INTERVIEW IN 6 QUESTIONS

```
Question 1: "Tell me about your project"
  └─ 1 min pitch about architecture

Question 2: "Walk me through the login flow"
  └─ 3-5 min detailed explanation

Question 3: "How do you handle errors?"
  └─ 2-3 min discussing status codes

Question 4: "What would you improve?"
  └─ 2-3 min discussing security/scalability

Question 5: "What do you want to learn?"
  └─ 1-2 min about Supabase (their stack)

Question 6: "Questions for us?"
  └─ 2-3 min asking about company
```

---

## ✨ QUICK VISUAL FACTS

```
╔══════════════════════════════════════════════════════════╗
║           YOUR PROJECT QUICK FACTS                       ║
╠══════════════════════════════════════════════════════════╣
║                                                          ║
║  Framework:    Next.js 13+ (App Router)                ║
║  Frontend:     React + TypeScript                       ║
║  Backend:      API Routes (Express-like)                ║
║  Database:     MongoDB + Mongoose                       ║
║  Auth:         Custom headers + role checks             ║
║  UI:           Radix UI + shadcn/ui + Tailwind         ║
║  Styling:      CSS + PostCSS                            ║
║  Deployment:   Vercel (serverless)                      ║
║  Users:        Admin, Trustee, Auditor, Viewer         ║
║  Features:     Dashboard, CRUD for all resources       ║
║                                                          ║
║  API Endpoints: ~15 routes with GET/POST/PUT/PATCH/DEL ║
║  Models:       User, Donor, Payment, Expenditure, etc  ║
║  Status Codes: 200, 201, 400, 401, 403, 404, 500       ║
║                                                          ║
╚══════════════════════════════════════════════════════════╝
```

---

## 🧠 MEMORY PALACE TECHNIQUE

**Memorize this path:**

```
START AT SIGN-IN ─┐
                  ├─ User enters credentials
                  │
                  ├─ POST /api/auth/login
                  │
                  ├─ ✓ Validate input (400 if not)
                  │
                  ├─ ✓ Query MongoDB for user
                  │
                  ├─ ✓ Check password (401 if wrong)
                  │
                  ├─ ✓ Update lastLogin
                  │
                  ├─ ✓ Return user + role (200)
                  │
                  ├─ Store role in browser
                  │
                  └─ Navigate to /dashboard
                     │
                     ├─ GET /api/dashboard/stats
                     │
                     ├─ ✓ Validate auth header (403 if not)
                     │
                     ├─ ✓ Connect to MongoDB
                     │
                     ├─ ✓ Query for data
                     │
                     ├─ ✓ Return stats (200)
                     │
                     └─ Render dashboard with data
```

---

**Use these visuals to reinforce your learning. Draw them on a whiteboard during the interview!** ✏️

---

*Visual Reference Created: May 3, 2026*
*For: Full-Stack Developer Interview Prep*
