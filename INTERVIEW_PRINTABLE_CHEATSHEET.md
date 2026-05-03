# 🖨️ PRINTABLE CHEAT SHEET
## Next.js Interview - Print This!

---

## PAGE 1: API ROUTES

```
┌─────────────────────────────────────────────────────────┐
│         API ROUTE BASIC STRUCTURE                       │
├─────────────────────────────────────────────────────────┤

File: app/api/[resource]/route.ts

import { NextRequest, NextResponse } from "next/server"

export async function GET(req: NextRequest) {
  return NextResponse.json({ data: [] })
}

export async function POST(req: NextRequest) {
  const body = await req.json()
  return NextResponse.json({ success: true }, { status: 201 })
}

export async function PUT(req: NextRequest, { params }) {
  const { id } = await params
  return NextResponse.json({ updated: true })
}

export async function DELETE(req: NextRequest, { params }) {
  const { id } = await params
  return NextResponse.json({ deleted: true })
}

┌─────────────────────────────────────────────────────────┐
│         HOW TO GET DATA FROM REQUEST                    │
├─────────────────────────────────────────────────────────┤

// Body
const body = await req.json()
const username = body.username

// Query params
const url = new URL(req.url)
const page = url.searchParams.get("page")

// Headers
const userRole = req.headers.get("x-user-role")

// Dynamic params (MUST AWAIT!)
export async function GET(req, { params }) {
  const { id } = await params
}

┌─────────────────────────────────────────────────────────┐
│         STATUS CODES & RESPONSES                        │
├─────────────────────────────────────────────────────────┤

200 OK:
  return NextResponse.json({ data })

201 Created:
  return NextResponse.json({ data }, { status: 201 })

400 Bad Request:
  return NextResponse.json({ error: "message" }, { status: 400 })

401 Unauthorized:
  return NextResponse.json({ error: "message" }, { status: 401 })

403 Forbidden:
  return NextResponse.json({ error: "message" }, { status: 403 })

404 Not Found:
  return NextResponse.json({ error: "message" }, { status: 404 })

500 Server Error:
  return NextResponse.json({ error: "message" }, { status: 500 })
```

---

## PAGE 2: YOUR ENDPOINTS

```
┌────────────────────────────────────────────────────────┐
│              YOUR API ENDPOINTS                        │
├────────────────────────────────────────────────────────┤

POST /api/auth/login
  Input: { username, password }
  Returns: { user: { id, username, role } }
  Status: 200 (success), 401 (wrong password)

GET /api/users (admin only)
  Returns: { users: [...] }
  Status: 200, 403 (not admin)

POST /api/users (admin only)
  Input: { name, email, role, password }
  Returns: { user: {...} }
  Status: 201 (created), 400 (invalid), 403

PATCH /api/users/[id] (admin only)
  Updates user
  Status: 200, 400, 403, 404

GET /api/donors
  Returns: { donors: [...] }
  Status: 200

POST /api/donors
  Input: { name, email, phone, status }
  Status: 201, 400

GET /api/donors/[id]
  Status: 200, 404

PUT /api/donors/[id]
  Status: 200, 400, 404

DELETE /api/donors/[id]
  Status: 200, 404

GET /api/payments
  Status: 200

POST /api/payments
  Status: 201, 400

DELETE /api/payments/[id]
  Status: 200, 404

GET /api/expenditures
  Status: 200

POST /api/expenditures
  Status: 201, 400

PUT /api/expenditures/[id]
  Status: 200, 400, 404

DELETE /api/expenditures/[id]
  Status: 200, 404

GET /api/dashboard/stats
  Query: ?fromDate=...&toDate=...
  Returns: Dashboard statistics
  Status: 200

GET /api/settings
  Status: 200

PUT /api/settings
  Status: 200, 400, 403

GET /api/reports/ledger
  Returns: Financial ledger report
  Status: 200
```

---

## PAGE 3: AUTH & VALIDATION

```
┌────────────────────────────────────────────────────────┐
│           AUTHENTICATION PATTERN                       │
├────────────────────────────────────────────────────────┤

Step 1: Login (public)
  const { username, password } = await req.json()
  if (!username || !password) return json({error}, 400)
  const user = await User.findOne({ username })
  if (!user) return json({error: "Not found"}, 404)
  if (user.password !== password) return json({error}, 401)
  return json({ user })

Step 2: Send role in header
  Browser stores user.role
  Sends: x-user-role: Admin (in headers)

Step 3: Verify on protected routes
  const role = req.headers.get("x-user-role")
  if (role !== "Admin") return json({error: "Forbidden"}, 403)

┌────────────────────────────────────────────────────────┐
│           VALIDATION PATTERNS                          │
├────────────────────────────────────────────────────────┤

Required field:
  const name = String(body.name || "").trim()
  if (!name) return json({error: "required"}, 400)

Enum validation:
  const roles = new Set(["Admin", "Trustee", "Auditor"])
  if (!roles.has(role)) return json({error: "invalid"}, 400)

Unique check:
  const exists = await User.exists({ username })
  if (exists) return json({error: "taken"}, 400)

Type validation:
  const amount = parseFloat(body.amount)
  if (isNaN(amount)) return json({error: "invalid"}, 400)

┌────────────────────────────────────────────────────────┐
│           ERROR HANDLING TEMPLATE                      │
├────────────────────────────────────────────────────────┤

export async function POST(req: NextRequest) {
  try {
    await connectDB()

    // 1. Validate input
    const body = await req.json()
    if (!body.name) {
      return NextResponse.json(
        { error: "Name required" },
        { status: 400 }
      )
    }

    // 2. Check auth
    if (req.headers.get("x-user-role") !== "Admin") {
      return NextResponse.json(
        { error: "Forbidden" },
        { status: 403 }
      )
    }

    // 3. Query database
    const doc = await Model.create(body)
    if (!doc) {
      return NextResponse.json(
        { error: "Failed to create" },
        { status: 500 }
      )
    }

    // 4. Return success
    return NextResponse.json({ data: doc }, { status: 201 })

  } catch (error) {
    console.error(error)
    return NextResponse.json(
      { error: "Server error" },
      { status: 500 }
    )
  }
}
```

---

## PAGE 4: HTTP STATUS CODES DECISION TREE

```
┌────────────────────────────────────────────────────────┐
│        STATUS CODE DECISION CHART                      │
├────────────────────────────────────────────────────────┤

1. Input validation failed? → 400 Bad Request
   Examples:
   - Missing required field
   - Invalid format (email, number)
   - Value not in allowed set

2. Authentication failed? → 401 Unauthorized
   Examples:
   - Wrong password
   - User not found
   - Session expired

3. Authenticated but not authorized? → 403 Forbidden
   Examples:
   - User not Admin but accessing admin route
   - Viewer trying to delete data

4. Resource not found? → 404 Not Found
   Examples:
   - GET /api/users/nonexistent-id
   - User trying to update deleted record

5. Creating resource successfully? → 201 Created
   Examples:
   - POST /api/users with valid input
   - POST /api/donors with valid input

6. Getting/updating data successfully? → 200 OK
   Examples:
   - GET /api/users
   - PUT /api/donors/[id]
   - PATCH /api/users/[id]

7. Server error/exception? → 500 Internal Server Error
   Examples:
   - Database connection failed
   - Unhandled exception in code

┌────────────────────────────────────────────────────────┐
│        YOUR CODE EXAMPLES                              │
├────────────────────────────────────────────────────────┤

400 - Missing fields:
  if (!password || !name) {
    return NextResponse.json(
      { error: "Missing required fields" },
      { status: 400 }
    )
  }

401 - Wrong password:
  if (user.password !== password) {
    return NextResponse.json(
      { error: "Invalid username or password" },
      { status: 401 }
    )
  }

403 - No permission:
  if (!assertAdmin(req)) {
    return NextResponse.json(
      { error: "Forbidden" },
      { status: 403 }
    )
  }

404 - Not found:
  if (!user) {
    return NextResponse.json(
      { error: "User not found" },
      { status: 404 }
    )
  }

201 - Created:
  return NextResponse.json(
    { user: userObject },
    { status: 201 }
  )

200 - Success:
  return NextResponse.json({ users })

500 - Server error:
  catch (error) {
    return NextResponse.json(
      { error: "Failed to create user" },
      { status: 500 }
    )
  }
```

---

## PAGE 5: DATABASE & MONGODB

```
┌────────────────────────────────────────────────────────┐
│        MONGODB CONNECTION PATTERN                      │
├────────────────────────────────────────────────────────┤

// Always connect first
import { connectDB } from "@/src/lib/mongodb"

export async function GET(req: NextRequest) {
  try {
    await connectDB()  // ← ESSENTIAL

    // Now query the database
    const data = await Model.find({})

    return NextResponse.json({ data })
  } catch (error) {
    return NextResponse.json({ error }, { status: 500 })
  }
}

┌────────────────────────────────────────────────────────┐
│        COMMON MONGODB OPERATIONS                       │
├────────────────────────────────────────────────────────┤

CREATE:
  const user = await User.create({
    name: "John",
    email: "john@example.com"
  })

READ (by ID):
  const user = await User.findById(id)

READ (by query):
  const user = await User.findOne({ username: "john" })

READ ALL:
  const users = await User.find({}).lean()
  const users = await User.find({}).sort({ name: 1 })

UPDATE:
  await User.findByIdAndUpdate(id, { name: "Jane" })

DELETE:
  await User.findByIdAndDelete(id)

COUNT:
  const count = await User.countDocuments()

EXISTS:
  const exists = await User.exists({ username })

┌────────────────────────────────────────────────────────┐
│        YOUR DATABASE MODELS                            │
├────────────────────────────────────────────────────────┤

User:
  id, username, password, name, email, role, status
  lastLogin, createdAt

Donor:
  id, name, email, phone, status

Payment:
  id, paymentId, donorId, amount, method
  date, month, year

Expenditure:
  id, category, amount, date, description

Property:
  id, name, address, value, type

AuditLog:
  id, action, model, recordId, description
  performedBy, timestamp
```

---

## PAGE 6: SUPABASE (THEIR STACK)

```
┌────────────────────────────────────────────────────────┐
│        SUPABASE BASICS                                 │
├────────────────────────────────────────────────────────┤

What is it?
  PostgreSQL + Authentication + Real-time + Storage

Difference from your MongoDB:
  Your project:        Supabase/PostgreSQL:
  - Document DB        - Relational DB
  - Custom auth        - Built-in auth
  - No real-time       - Real-time subscriptions
  - No RLS             - RLS (row-level security)

Setup:
  npm install @supabase/supabase-js

Client-side:
  import { createClient } from '@supabase/supabase-js'
  const supabase = createClient(URL, ANON_KEY)
  const { data } = await supabase.from('users').select('*')

Server-side:
  import { createRouteHandlerClient } from '@supabase/auth-helpers-nextjs'
  const supabase = createRouteHandlerClient({ cookies })
  const { data } = await supabase.from('users').select('*')

Authentication:
  // Sign up
  await supabase.auth.signUp({ email, password })

  // Sign in
  await supabase.auth.signIn({ email, password })

  // Get user
  const user = await supabase.auth.user()

  // Sign out
  await supabase.auth.signOut()

Real-time:
  supabase
    .from('donors')
    .on('*', (payload) => {
      console.log('Change:', payload.new)
    })
    .subscribe()

RLS (Row Level Security):
  CREATE POLICY "Users see own data"
  ON donors FOR SELECT
  USING (auth.uid() = user_id);

Environment Variables:
  NEXT_PUBLIC_SUPABASE_URL
  NEXT_PUBLIC_SUPABASE_ANON_KEY
  SUPABASE_SERVICE_ROLE_KEY

┌────────────────────────────────────────────────────────┐
│        HOW TO ANSWER SUPABASE QUESTIONS                │
├────────────────────────────────────────────────────────┤

Q: "Have you used Supabase?"
A: "Not in production, but I understand it well.
   It's PostgreSQL + Auth + Real-time.
   I'm excited to learn it for this role."

Q: "How would you migrate to Supabase?"
A: "1. Map MongoDB collections to PostgreSQL tables
    2. Create migration script for existing data
    3. Update API routes to use Supabase client
    4. Implement RLS policies for security
    5. Test thoroughly before cutover"

Q: "What's the advantage of RLS?"
A: "Security at the database level, not just in code.
   Even if API is compromised, RLS protects data.
   Policies like: 'Users see only own data'"

Q: "How would you do real-time updates?"
A: "With Supabase WebSocket subscriptions:
   supabase.from('donors').on('*', handler).subscribe()
   Much better than polling every N seconds."
```

---

## PAGE 7: YOUR PITCH & TOP QUESTIONS

```
┌────────────────────────────────────────────────────────┐
│        YOUR 30-SECOND PITCH                            │
├────────────────────────────────────────────────────────┤

"I built a full-stack dashboard using Next.js for both
frontend and API, MongoDB for the database, and deployed
it on Vercel. The application manages donors, payments,
and expenditures with role-based access control. All API
endpoints return proper HTTP status codes and follow RESTful
principles. I'm excited to apply this foundation to
your tech stack with Supabase."

┌────────────────────────────────────────────────────────┐
│        ANSWER YOUR TOP 5 QUESTIONS                     │
├────────────────────────────────────────────────────────┤

Q1: "Walk me through login"
A: User submits username/password → POST /api/auth/login
   → Validate input (400 if missing) → Query MongoDB
   → Compare password (401 if wrong) → Return user+role
   → Browser stores role and sends as header in future requests

Q2: "How do you handle authorization?"
A: Custom header-based role checking. Check the x-user-role
   header, if not Admin return 403 Forbidden. In production,
   I'd use JWT tokens for better security.

Q3: "What HTTP status codes do you use?"
A: 200 OK (success), 201 Created (POST), 400 Bad Request
   (invalid input), 401 Unauthorized (wrong password),
   403 Forbidden (no permission), 404 Not Found (missing
   resource), 500 Server Error (unhandled exception)

Q4: "What would you improve?"
A: Add Bcrypt for password hashing (currently plaintext)
   Use JWT tokens instead of headers, Add rate limiting,
   Implement proper logging, Add test coverage.

Q5: "Questions about Supabase?"
A: How do you use RLS policies for complex permissions?
   How does real-time perform at scale?
   Any migration challenges from MongoDB?

┌────────────────────────────────────────────────────────┐
│        CONFIDENCE BOOSTERS                             │
├────────────────────────────────────────────────────────┤

Remember:
  ✓ You built a real, working application
  ✓ You understand every piece of your code
  ✓ You know HTTP semantics deeply
  ✓ You've identified improvements (shows maturity)
  ✓ You're prepared for common questions

If you freeze:
  - Pause and breathe
  - Draw a diagram
  - "Let me think about that..."
  - It's OK to say "I'd research that"

You're hired because:
  1. Can you code? Yes (your project)
  2. Do you understand it? Yes (you built it)
  3. Can you grow? Yes (you identified improvements)
```

---

## PAGE 8: QUICK FACTS

```
┌────────────────────────────────────────────────────────┐
│        YOUR PROJECT QUICK FACTS                        │
├────────────────────────────────────────────────────────┤

✓ Framework:       Next.js 13+ (App Router)
✓ Frontend:        React + TypeScript
✓ Backend:         API Routes
✓ Database:        MongoDB + Mongoose
✓ Authentication:  Custom headers + role-based
✓ UI Components:   Radix UI + shadcn/ui
✓ Styling:         Tailwind CSS + PostCSS
✓ Deployment:      Vercel (serverless)
✓ Roles:           Admin, Trustee, Auditor, Viewer
✓ API Endpoints:   ~15 routes (GET/POST/PUT/PATCH/DELETE)
✓ Status Codes:    200, 201, 400, 401, 403, 404, 500

┌────────────────────────────────────────────────────────┐
│        MEMORY TRICKS                                   │
├────────────────────────────────────────────────────────┤

HTTP Status Codes:
  2XX = Happy (success)
  4XX = Your fault (bad request, not found, etc)
  5XX = Our fault (server error)

Authorization Logic:
  IF input is bad → 400
  IF password wrong → 401
  IF no permission → 403
  IF not found → 404
  IF server error → 500

API Handler Steps:
  1. Connect to DB
  2. Extract data from request
  3. Validate input (400 if bad)
  4. Check authorization (403 if not allowed)
  5. Query database (404 if not found)
  6. Return success with proper status

┌────────────────────────────────────────────────────────┐
│        IF YOU HAVE 2 MINUTES                           │
├────────────────────────────────────────────────────────┤

Read this:
  My project: Next.js frontend + API + MongoDB
  Tech choice: Unified stack, easy deployment
  Authentication: Login → store role → check header
  Status codes: Appropriate for each error type
  Improvement: Would add Bcrypt, JWT, logging

Say this:
  "Full-stack Next.js app with role-based access,
   proper HTTP semantics, MongoDB persistence,
   deployed on Vercel. I know what to improve."

Ask this:
  "How do you approach scaling?
   What's your monitoring strategy?
   Supabase vs other databases?"

Good luck! 🚀
```

---

**PRINT THIS DOCUMENT AND BRING TO INTERVIEW!**

*One page for each critical concept*
*Reference quickly between questions*
*Show confidence with concrete examples*

---

*Printable Cheat Sheet Created: May 3, 2026*
*For: Full-Stack Developer Interview*
*Print double-sided to save paper!*
