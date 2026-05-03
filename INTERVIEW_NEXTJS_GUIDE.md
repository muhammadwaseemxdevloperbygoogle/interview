# 🚀 Next.js Interview Preparation Guide
## Full-Stack Developer Preparation (Next.js + Supabase + Vercel)

**Created:** May 3, 2026  
**Duration:** 4 Hours  
**Tech Stack:** Next.js, MongoDB, Clerk/Auth, Vercel  

---

## 📋 TABLE OF CONTENTS
1. [Next.js Fundamentals](#nextjs-fundamentals)
2. [Project Architecture](#project-architecture)
3. [API Routes & Handlers](#api-routes--handlers)
4. [Authentication & Authorization](#authentication--authorization)
5. [Database Integration](#database-integration)
6. [HTTP Status Codes](#http-status-codes)
7. [Key Patterns in Your Code](#key-patterns-in-your-code)
8. [Common Interview Questions](#common-interview-questions)
9. [Performance & Deployment](#performance--deployment)

---

## NEXT.JS FUNDAMENTALS

### 1. **App Router vs Pages Router**
- **Your Project Uses:** App Router (`/app` directory)
- **Key Difference:** App Router is the new standard, Pages Router is legacy
- **Benefits of App Router:**
  - Nested layouts and routing
  - Server components by default
  - Built-in data fetching improvements
  - Streaming and Suspense support

### 2. **File Structure Understanding**

```
app/
├── page.tsx           ← Root page (GET / redirects to /sign-in)
├── layout.tsx         ← Root layout wrapper
├── globals.css        ← Global styles
├── api/               ← API Routes
│   ├── auth/
│   │   ├── login/
│   │   │   └── route.ts
│   │   └── verify-password/
│   │       └── route.ts
│   ├── users/
│   │   ├── route.ts   ← GET /api/users, POST /api/users
│   │   └── [id]/
│   │       └── route.ts ← GET, PATCH /api/users/[id]
│   └── ...other routes
├── dashboard/
│   ├── layout.tsx
│   ├── page.tsx
│   └── [section]/     ← Dynamic routes
├── sign-in/
│   └── page.tsx
```

**Key Concepts:**
- `route.ts` files export HTTP handlers (GET, POST, PUT, PATCH, DELETE)
- `[id]` = dynamic segment (captured in params)
- `[[...slug]]` = optional catch-all routes
- `layout.tsx` = shared UI across routes
- Root `layout.tsx` wraps entire app

### 3. **Server vs Client Components**

```typescript
// SERVER COMPONENT (default in /app)
export default function ServerComponent() {
  // Direct DB access ✓
  // Secrets safe ✓
  // Can't use hooks ✗
  return <div>Server content</div>
}

// CLIENT COMPONENT
"use client"
import { useState } from "react"

export default function ClientComponent() {
  const [state, setState] = useState(null)  // ✓ Hooks work
  // Can't access DB directly ✗
  // Secrets exposed ✗
  return <div>Client content</div>
}
```

**Your Project:** Uses `"use client"` in [lib/auth-context.tsx](lib/auth-context.tsx#L1) for auth state management.

### 4. **Dynamic Routes**

Your code shows how to handle dynamic segments:

```typescript
// File: src/app/api/donors/[id]/route.ts
type Params = {
  params: Promise<{ id: string }>  // ← Must be Promise now!
}

export async function GET(_req: NextRequest, { params }: Params) {
  const { id } = await params  // ← Must await!
  // Use id...
}
```

**Important:** Next.js 13.5+ requires `params` to be a `Promise` that you must `await`.

---

## PROJECT ARCHITECTURE

### Your Tech Stack Breakdown

| Layer | Technology | Usage |
|-------|-----------|-------|
| **Frontend** | React + TypeScript | UI Components, State Management |
| **Routing** | Next.js App Router | Server-side routing, nested layouts |
| **Backend** | Next.js API Routes | REST API endpoints |
| **Database** | MongoDB + Mongoose | Data persistence |
| **Auth** | Custom + Clerk headers | User authentication, role-based access |
| **UI Components** | Radix UI + shadcn/ui | Pre-built accessible components |
| **Styling** | Tailwind CSS + PostCSS | Utility-first CSS |
| **Deployment** | Vercel | Hosting & CI/CD |

### Architecture Pattern: Layered Structure

```
Request Flow:
Client → Next.js API Route → MongoDB → Response

Example:
1. POST /api/users
2. Verify admin role via headers
3. Validate input
4. Create document in MongoDB
5. Return 201 with created user
```

---

## API ROUTES & HANDLERS

### 1. **Basic Structure**

```typescript
// src/app/api/payments/route.ts
import { NextRequest, NextResponse } from "next/server"

export async function GET(req: NextRequest) {
  try {
    // Your logic
    return NextResponse.json({ data: [] })
  } catch (error) {
    return NextResponse.json({ error: "message" }, { status: 500 })
  }
}

export async function POST(req: NextRequest) {
  try {
    const body = await req.json()
    // Validate
    // Create
    return NextResponse.json({ success: true }, { status: 201 })
  } catch (error) {
    return NextResponse.json({ error: "message" }, { status: 400 })
  }
}
```

### 2. **Accessing Request Data**

```typescript
// Query parameters
const searchParams = new URL(req.url).searchParams
const fromDate = searchParams.get("fromDate")

// Headers
const userRole = req.headers.get("x-user-role")

// Body (JSON)
const body = await req.json()

// URL params (dynamic routes)
const { id } = await params
```

**Your Project Example:** [src/app/api/dashboard/stats/route.ts](src/app/api/dashboard/stats/route.ts#L27) uses `searchParams` for date filtering.

### 3. **API Response Patterns**

```typescript
// Success
return NextResponse.json({ data }, { status: 200 })

// Created (POST)
return NextResponse.json({ user }, { status: 201 })

// Error - Bad Request
return NextResponse.json({ error: "message" }, { status: 400 })

// Error - Unauthorized
return NextResponse.json({ error: "message" }, { status: 401 })

// Error - Forbidden
return NextResponse.json({ error: "message" }, { status: 403 })

// Error - Not Found
return NextResponse.json({ error: "message" }, { status: 404 })

// Error - Server Error
return NextResponse.json({ error: "message" }, { status: 500 })
```

### 4. **Your API Routes Summary**

| Endpoint | Method | Purpose | Status Codes Used |
|----------|--------|---------|------------------|
| `/api/auth/login` | POST | User authentication | 200, 400, 401, 500 |
| `/api/auth/verify-password` | POST | Password verification | 200, 400, 401, 404, 500 |
| `/api/users` | GET | List all users (admin) | 200, 403, 500 |
| `/api/users` | POST | Create new user (admin) | 201, 400, 403, 500 |
| `/api/users/[id]` | PATCH | Update user | 200, 400, 403, 404, 500 |
| `/api/donors` | GET, POST | Donor CRUD | 200, 201, 400, 500 |
| `/api/donors/[id]` | GET, PUT, DELETE | Single donor ops | 200, 400, 404, 500 |
| `/api/payments` | GET, POST | Payment CRUD | 200, 201, 400, 500 |
| `/api/expenditures` | GET, POST | Expense CRUD | 200, 201, 400, 500 |
| `/api/properties` | GET, POST | Property CRUD | 200, 201, 400, 500 |
| `/api/settings` | GET, PUT | App settings | 200, 400, 500 |
| `/api/dashboard/stats` | GET | Dashboard stats | 200, 500 |
| `/api/reports/ledger` | GET | Financial reports | 200, 500 |

---

## AUTHENTICATION & AUTHORIZATION

### 1. **Custom Authentication Pattern**

Your project uses **header-based role verification**:

```typescript
// src/app/api/users/route.ts
function assertAdmin(req: NextRequest) {
  return req.headers.get("x-user-role") === "Admin"
}

export async function POST(req: NextRequest) {
  if (!assertAdmin(req)) {
    return NextResponse.json({ error: "Forbidden" }, { status: 403 })
  }
  // Admin-only logic
}
```

**Flow:**
1. Client makes request with `x-user-role` header
2. API checks role in header
3. Grants/denies access based on role
4. Returns 403 if unauthorized

### 2. **Role-Based Access Control (RBAC)**

```typescript
const allowedRoles = new Set(["Admin", "Trustee", "Auditor", "Viewer"])

// Validate role
if (!allowedRoles.has(role)) {
  return NextResponse.json({ error: "Invalid role" }, { status: 400 })
}
```

**Roles in Your Project:**
- **Admin** - Full access to users, settings
- **Trustee** - Can manage donors, payments
- **Auditor** - Read-only access to reports
- **Viewer** - Limited view access

### 3. **Authentication Patterns**

```typescript
// Login endpoint - src/app/api/auth/login/route.ts
export async function POST(req: NextRequest) {
  const { username, password } = await req.json()
  
  // 1. Validate input
  if (!username || !password) {
    return NextResponse.json({ error: "Required fields" }, { status: 400 })
  }
  
  // 2. Find user
  const user = await User.findOne({ username, status: "Active" })
  if (!user || user.password !== password) {
    return NextResponse.json({ error: "Invalid credentials" }, { status: 401 })
  }
  
  // 3. Update last login
  user.lastLogin = new Date()
  await user.save()
  
  // 4. Return user data
  return NextResponse.json({ user: { id, username, role, ... } })
}
```

### 4. **Security Considerations**

⚠️ **Your Project:**
```typescript
// SECURITY ISSUE: Plain text passwords
user.password !== password  // ✗ Not hashed!
```

**For Interview:** Be ready to discuss:
- Bcrypt for password hashing
- JWT tokens for stateless auth
- HTTPS/TLS for secure transmission
- CORS for cross-origin requests
- Environment variables for secrets

---

## DATABASE INTEGRATION

### 1. **MongoDB Connection Pattern**

```typescript
// src/lib/mongodb.ts
import { connectDB } from "@/src/lib/mongodb"

export async function GET(req: NextRequest) {
  try {
    await connectDB()  // ← Always connect first
    
    // Database operations
    const data = await Model.find({})
    
    return NextResponse.json({ data })
  } catch (error) {
    return NextResponse.json({ error: "DB error" }, { status: 500 })
  }
}
```

### 2. **Mongoose Models**

Your project has these models:

```typescript
// src/models/User.ts
interface User {
  id: string
  username: string
  password: string
  name: string
  email?: string
  role: "Admin" | "Trustee" | "Auditor" | "Viewer"
  status: "Active" | "Inactive"
  lastLogin?: Date
  groups?: string[]
  rights?: string[]
}

// src/models/Donor.ts
interface Donor {
  id: string
  name: string
  email?: string
  phone?: string
  status: "Active" | "Inactive"
  // ... other fields
}

// src/models/Payment.ts
interface Payment {
  paymentId: string
  donorId: string
  method: "cash" | "bank_transfer" | "easypaisa" | "jazzcash"
  amount: number
  date: Date
  month: number
  year: number
}

// src/models/Expenditure.ts
interface Expenditure {
  id: string
  category: string
  amount: number
  date: Date
  description: string
}
```

### 3. **Common Database Operations**

```typescript
// CREATE
const user = await User.create({ username, password, ... })

// READ
const user = await User.findById(id)
const users = await User.find({}).lean()
const user = await User.findOne({ username })

// UPDATE
await User.findByIdAndUpdate(id, { name: "New Name" })

// DELETE
await User.findByIdAndDelete(id)

// COUNT
const count = await Donor.countDocuments()

// AGGREGATE
const stats = await Donor.aggregate([...])

// TRANSACTIONS/JOINS
const donor = await Donor.findById(id)
const payments = await Payment.find({ donorId: id })
```

### 4. **Data Validation**

```typescript
// Input validation pattern from your code
const userId = String(body.userId || "").trim()
if (!userId) {
  return NextResponse.json({ error: "required" }, { status: 400 })
}
```

---

## HTTP STATUS CODES

### Complete Reference for Your Project

#### **2xx Success Codes**

| Code | Name | Usage | Your Code |
|------|------|-------|-----------|
| **200** | OK | Successful GET, PUT | Dashboard stats GET |
| **201** | Created | Resource created via POST | User creation POST |
| **204** | No Content | Success, no body | (Not used in your code) |

#### **4xx Client Error Codes**

| Code | Name | Usage | Your Code |
|------|------|-------|-----------|
| **400** | Bad Request | Invalid input | Missing fields, invalid role |
| **401** | Unauthorized | Missing/invalid auth | Invalid password |
| **403** | Forbidden | Valid auth but insufficient perms | Non-admin accessing admin route |
| **404** | Not Found | Resource doesn't exist | User not found, payment not found |
| **409** | Conflict | Duplicate (uniqueness violation) | (Not explicitly used) |

#### **5xx Server Error Codes**

| Code | Name | Usage | Your Code |
|------|------|-------|-----------|
| **500** | Internal Server Error | Unhandled exception | Database errors |
| **503** | Service Unavailable | DB unavailable | MongoDB connection fails |

### Status Code Examples from Your Code

```typescript
// 200 - Success
return NextResponse.json({ users })

// 201 - Created
return NextResponse.json({ user: userObject }, { status: 201 })

// 400 - Bad Request
if (!password || !name) {
  return NextResponse.json({ error: "Missing required fields" }, { status: 400 })
}

// 401 - Unauthorized
if (user.password !== password) {
  return NextResponse.json({ error: "Invalid username or password" }, { status: 401 })
}

// 403 - Forbidden
if (!assertAdmin(req)) {
  return NextResponse.json({ error: "Forbidden" }, { status: 403 })
}

// 404 - Not Found
if (!user) {
  return NextResponse.json({ error: "User not found" }, { status: 404 })
}

// 500 - Server Error
catch (error) {
  return NextResponse.json({ error: "Failed to create user" }, { status: 500 })
}
```

### Decision Tree for Status Codes

```
Request received
│
├─ Invalid input? → 400 Bad Request
├─ Missing auth? → 401 Unauthorized
├─ Auth valid but forbidden? → 403 Forbidden
├─ Resource not found? → 404 Not Found
├─ Creating resource? → 201 Created
├─ Updating/getting? → 200 OK
├─ Server error? → 500 Internal Server Error
```

---

## KEY PATTERNS IN YOUR CODE

### 1. **Dynamic Routes with Params (Promise)**

```typescript
// IMPORTANT: Next.js 13.5+ requires this pattern
type Params = {
  params: Promise<{ id: string }>
}

export async function PUT(req: NextRequest, { params }: Params) {
  const { id } = await params  // ← MUST await!
  // Use id
}
```

### 2. **Force Dynamic Rendering**

```typescript
// app/api/users/route.ts
export const dynamic = 'force-dynamic'
export * from '@/src/app/api/users/route'
```

**Why?** Forces Next.js to treat routes as dynamic (not cached), ensuring fresh data on every request.

### 3. **Input Normalization**

```typescript
// Normalize string input
const username = String(body.username || "").trim().toLowerCase()

// Normalize arrays
function normalizeStringArray(value: unknown) {
  return Array.isArray(value)
    ? value.map((item) => String(item).trim()).filter(Boolean)
    : []
}
```

### 4. **Slug Generation**

```typescript
function slugifyUsername(value: string) {
  return value
    .trim()
    .toLowerCase()
    .replace(/[^a-z0-9]+/g, "-")
    .replace(/^-+|-+$/g, "")
}
// "John Doe" → "john-doe"
```

### 5. **Unique Value Generation**

```typescript
async function getUniqueUsername(preferredUsername: string, fallbackName: string) {
  const base = slugifyUsername(preferredUsername || fallbackName) || "user"
  let candidate = base
  let suffix = 1

  while (await User.exists({ username: candidate })) {
    suffix += 1
    candidate = `${base}-${suffix}`
  }
  return candidate
}
// "john-doe" → "john-doe-1" → "john-doe-2" ...
```

### 6. **Error Handling**

```typescript
// Extract error message safely
if (typeof error === "object" && error !== null) {
  const maybeError = error as { code?: number; message?: string }
  if (maybeError.message) {
    return NextResponse.json({ error: maybeError.message }, { status: 500 })
  }
}
```

### 7. **Audit Logging**

```typescript
await AuditLog.create({
  action: "delete",
  model: "Payment",
  recordId: payment.paymentId,
  description: "Payment deleted",
  performedBy: "Admin",
})
```

**Pattern:** Track all changes for compliance & debugging.

### 8. **Conditional Rendering in API**

```typescript
// src/app/api/users/[id]/route.ts
const update: Record<string, unknown> = {}
if (body.name !== undefined) update.name = String(body.name).trim()
if (body.email !== undefined) update.email = body.email ? String(body.email).trim().toLowerCase() : undefined
// Only include fields that were actually provided
```

---

## COMMON INTERVIEW QUESTIONS

### 1. **"Explain your project architecture. What tech did you choose and why?"**

**Answer Template:**
> "We built a full-stack dashboard using Next.js for both frontend and API. Here's why:
> 
> - **Next.js App Router:** Unified frontend/backend, simpler deployment, built-in optimizations
> - **MongoDB + Mongoose:** Flexible schema, great for rapid iteration
> - **Custom Auth:** Simple role-based access via headers (though in production we'd use JWT + Bcrypt)
> - **Vercel:** Zero-config deployment, serverless functions, perfect for Next.js
> 
> The architecture is layered:
> ```
> React Components → Next.js API Routes → MongoDB
> ```
> API routes validate input, handle auth, connect to DB, return proper status codes."

### 2. **"Walk me through the login flow."**

**Answer:**
> "1. User fills username/password in sign-in form
> 2. Form sends POST to `/api/auth/login` with credentials
> 3. API route:
>    - Validates input (400 if missing)
>    - Queries User document
>    - Compares password (401 if invalid)
>    - Updates lastLogin timestamp
>    - Returns user object with role
> 4. Frontend stores user data + role in auth context
> 5. Role is sent as `x-user-role` header in subsequent requests
> 6. API routes check this header for authorization"

### 3. **"How do you handle authorization?"**

**Answer:**
> "We use header-based role checking:
> 
> ```typescript
> function assertAdmin(req: NextRequest) {
>   return req.headers.get("x-user-role") === "Admin"
> }
> 
> if (!assertAdmin(req)) {
>   return NextResponse.json({ error: "Forbidden" }, { status: 403 })
> }
> ```
> 
> This pattern is applied to sensitive routes like user creation, settings updates.
> 
> **Production improvements:**
> - Store roles in JWT token (not header)
> - Use middleware to validate auth on protected routes
> - Implement proper RBAC with permissions matrix"

### 4. **"Why use `await params` instead of `params` directly?"**

**Answer:**
> "Starting with Next.js 13.5, params is a Promise to support streaming. We must await it:
> 
> ```typescript
> type Params = {
>   params: Promise<{ id: string }>
> }
> 
> export async function GET(_req, { params }: Params) {
>   const { id } = await params  // ← Required!
> }
> ```
> 
> This allows Next.js to stream pages and gradually load data."

### 5. **"What HTTP status codes did you use and when?"**

**Answer:**
> "- **200 OK:** Successful GET, PUT, PATCH (retrieving/updating data)
> - **201 Created:** POST that creates a resource
> - **400 Bad Request:** Validation failed (missing fields, invalid role)
> - **401 Unauthorized:** Auth required but failed (wrong password)
> - **403 Forbidden:** Auth valid but insufficient permissions
> - **404 Not Found:** Resource doesn't exist
> - **500 Internal Server Error:** Unhandled exceptions
> 
> Example from our code:
> ```typescript
> if (!username || !password) return NextResponse.json({...}, {status: 400})
> if (!user) return NextResponse.json({...}, {status: 404})
> if (user.password !== password) return NextResponse.json({...}, {status: 401})
> ```"

### 6. **"How do you validate data in your API routes?"**

**Answer:**
> "Multi-layered validation:
> 
> 1. **Input Conversion:**
>    ```typescript
>    const username = String(body.username || "").trim().toLowerCase()
>    ```
> 
> 2. **Required Field Check:**
>    ```typescript
>    if (!username || !password) {
>      return NextResponse.json({error: "Missing fields"}, {status: 400})
>    }
>    ```
> 
> 3. **Type/Enum Validation:**
>    ```typescript
>    const allowedRoles = new Set(["Admin", "Trustee", "Auditor", "Viewer"])
>    if (!allowedRoles.has(role)) {
>      return NextResponse.json({error: "Invalid role"}, {status: 400})
>    }
>    ```
> 
> 4. **Database Check (unique values):**
>    ```typescript
>    if (await User.exists({username})) {
>      return NextResponse.json({error: "Username taken"}, {status: 400})
>    }
>    ```
> 
> **Better approach:** Use `zod` or `joi` for schema validation."

### 7. **"What's the difference between server and client components?"**

**Answer:**
> "**Server Components (default in /app):**
> - Run on Vercel server
> - Can access databases/secrets directly
> - Can't use hooks (useState, useContext, etc)
> - Zero JavaScript sent to browser
> - Great for data fetching
> 
> **Client Components (`"use client"`):**
> - Run in user's browser
> - Can use React hooks
> - Can't access databases directly
> - JavaScript bundle sent to client
> - Great for interactivity
> 
> Our project uses:
> ```typescript
> // Server component (default)
> export default function DashboardPage() {
>   const data = await fetchData()  // ✓ Can do this
>   return <ClientComponent data={data} />
> }
> 
> // Client component
> 'use client'
> export function ClientComponent({ data }) {
>   const [filtered, setFiltered] = useState(data)  // ✓ Can do this
> }
> ```"

### 8. **"How would you improve the security of your auth system?"**

**Answer:**
> "Current issues and fixes:
> 
> 1. **Plain text passwords:**
>    ```typescript
>    // ✗ Current: user.password !== password
>    // ✓ Better: bcrypt.compare(password, user.passwordHash)
>    ```
> 
> 2. **Header-based auth (not stateless):**
>    ```typescript
>    // ✗ Current: req.headers.get("x-user-role")
>    // ✓ Better: JWT token with signature
>    ```
> 
> 3. **No HTTPS enforcement:**
>    - Vercel forces HTTPS automatically ✓
> 
> 4. **No rate limiting:**
>    ```typescript
>    // Add: Check IP + timestamp to prevent brute force
>    ```
> 
> 5. **No session timeout:**
>    ```typescript
>    // Add: Check token expiration
>    ```"

### 9. **"How would you handle errors in production?"**

**Answer:**
> "Current approach:
> ```typescript
> catch (error) {
>   return NextResponse.json({error: "Failed to create user"}, {status: 500})
> }
> ```
> 
> Production improvements:
> 
> 1. **Error logging:**
>    ```typescript
>    catch (error) {
>      logger.error('User creation failed', { error, userId: body.userId })
>      return NextResponse.json({error: "Server error"}, {status: 500})
>    }
>    ```
> 
> 2. **Error tracking (Sentry):**
>    ```typescript
>    Sentry.captureException(error)
>    ```
> 
> 3. **Don't expose internal errors to client**
> 
> 4. **Distinguish error types (validation vs system)**"

### 10. **"How would you test your API routes?"**

**Answer:**
> "**Unit Tests (Jest):**
> ```typescript
> describe('POST /api/users', () => {
>   it('should create user with valid input', async () => {
>     const res = await POST(req, {params: Promise.resolve({})})
>     expect(res.status).toBe(201)
>   })
> 
>   it('should return 400 for missing fields', async () => {
>     const res = await POST(invalidReq, {...})
>     expect(res.status).toBe(400)
>   })
> })
> ```
> 
> **Integration Tests (Supertest):**
> ```typescript
> it('should create and retrieve user', async () => {
>   const user = await createUser({...})
>   const res = await request(app).get(`/api/users/${user.id}`)
>   expect(res.body.user.id).toBe(user.id)
> })
> ```
> 
> **E2E Tests (Playwright/Cypress):**
> ```typescript
> test('login flow', async ({page}) => {
>   await page.goto('/sign-in')
>   await page.fill('input[name=username]', 'admin')
>   await page.click('button[type=submit]')
>   await expect(page).toHaveURL('/dashboard')
> })
> ```"

---

## PERFORMANCE & DEPLOYMENT

### 1. **Next.js Performance Features (Your Project)**

```typescript
// Image optimization (disabled in your config)
// images: { unoptimized: true }

// API Route patterns
export const dynamic = 'force-dynamic'  // Prevents caching
export const revalidate = 3600           // ISR: revalidate every hour
```

### 2. **Vercel Deployment**

**Your setup is ready for Vercel:**
- ✓ Next.js app structure
- ✓ API routes (serverless functions)
- ✓ Environment variables (`.env.local`)
- ✓ TypeScript configuration
- ✓ `next.config.mjs` for customization

**Deployment command:**
```bash
vercel deploy
```

**Environment variables needed:**
```
MONGODB_URI=your_connection_string
CLERK_SECRET_KEY=your_clerk_key
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=...
```

### 3. **Database Connection Optimization**

```typescript
// src/lib/mongodb.ts
// Should implement connection pooling:
let cached = global.mongoose
if (!cached) cached = global.mongoose = { conn: null, promise: null }

export async function connectDB() {
  if (cached.conn) return cached.conn  // Reuse existing
  if (!cached.promise) {
    cached.promise = mongoose.connect(MONGODB_URI)
  }
  cached.conn = await cached.promise
  return cached.conn
}
```

### 4. **Caching Strategy**

| Type | Strategy | Use Case |
|------|----------|----------|
| **Dynamic Routes** | `export const dynamic = 'force-dynamic'` | Always fresh |
| **Static Routes** | Default (cached) | Dashboard layouts |
| **ISR** | `export const revalidate = 3600` | Reports (hourly refresh) |

### 5. **Monitoring**

- **Vercel Analytics:** [embedded in your code](package.json#L33) with `@vercel/analytics`
- **Database:** Monitor MongoDB connection pooling
- **Error Tracking:** Set up Sentry/LogRocket

---

## 🎯 LAST-MINUTE CHECKLIST (30 mins before interview)

### Know These Concepts Cold:
- [ ] App Router vs Pages Router (you use App Router)
- [ ] Server vs Client components and when to use each
- [ ] API routes with GET, POST, PUT, PATCH, DELETE
- [ ] Dynamic routes with `[id]` and `await params`
- [ ] All HTTP status codes: 200, 201, 400, 401, 403, 404, 500
- [ ] Authentication flow (login → validate → store → check headers)
- [ ] Authorization pattern (role checking)
- [ ] MongoDB/Mongoose basic operations
- [ ] Vercel deployment process
- [ ] Your project's actual structure and features

### Be Ready to:
- [ ] Draw architecture diagram (Frontend → API → DB)
- [ ] Explain login flow end-to-end
- [ ] Code a simple API route from scratch
- [ ] Explain why a status code was chosen
- [ ] Discuss security improvements needed
- [ ] Explain your code choices and trade-offs

### Have Ready:
- [ ] Your project repo link
- [ ] Specific files to point to (API examples)
- [ ] 2-3 questions about Supabase integration (they mentioned it!)
- [ ] Questions about their tech stack

### Practice Answering:
> "What's one thing you'd do differently if you rebuilt this project?"
> "What was the hardest part of this project?"
> "How would you scale this to 1M users?"
> "Walk me through your most complex feature"

---

## 📚 QUICK REFERENCE: YOUR STATUS CODE USAGE

```typescript
// PATTERNS YOU USE
200: return NextResponse.json({ data })
201: return NextResponse.json({ user }, { status: 201 })
400: return NextResponse.json({ error: msg }, { status: 400 })
401: return NextResponse.json({ error: msg }, { status: 401 })
403: return NextResponse.json({ error: "Forbidden" }, { status: 403 })
404: return NextResponse.json({ error: msg }, { status: 404 })
500: return NextResponse.json({ error: msg }, { status: 500 })

// DECISION LOGIC
Missing input? → 400
Wrong password? → 401
Don't have permission? → 403
Resource missing? → 404
Server crashed? → 500
```

---

## 🚀 GOOD LUCK!

You've built a solid full-stack project. Focus on:
1. **Explaining your actual code** (they'll ask)
2. **Understanding why you made choices** (they'll probe)
3. **Recognizing what could be better** (shows growth mindset)
4. **Connecting concepts** (API status codes → error handling → validation)

The fact that you're preparing with your actual codebase puts you ahead! 💪

**Remember:** The interview isn't just about what you know—it's about how you think and communicate. Good luck! 🎉

---

*Last Updated: May 3, 2026*
*For: Full-Stack Developer Interview*
*Tech Stack: Next.js, MongoDB, Clerk, Vercel*
