# ⚡ QUICK REFERENCE CHEAT SHEET
## Next.js Interview - 30 Minutes Before

---

## 1️⃣ API ROUTES STRUCTURE (5 mins)

```typescript
// File: app/api/[resource]/route.ts
import { NextRequest, NextResponse } from "next/server"

// Handlers - automatically matched to HTTP methods
export async function GET(req: NextRequest) { }
export async function POST(req: NextRequest) { }
export async function PUT(req: NextRequest) { }
export async function PATCH(req: NextRequest) { }
export async function DELETE(req: NextRequest) { }

// Dynamic routes: app/api/[resource]/[id]/route.ts
type Params = { params: Promise<{ id: string }> }
export async function GET(req: NextRequest, { params }: Params) {
  const { id } = await params  // ← MUST AWAIT!
}
```

---

## 2️⃣ HTTP STATUS CODES DECISION TREE (5 mins)

```
POST request?
├─ Validation failed? → 400 Bad Request
├─ Created resource? → 201 Created ✓
└─ Auth failed? → 401 Unauthorized

GET/PUT/PATCH request?
├─ Resource not found? → 404 Not Found
├─ Success? → 200 OK ✓
└─ No permission? → 403 Forbidden

Any request error?
└─ Server crash? → 500 Internal Server Error
```

**YOUR STATUS CODES:**
- `200` - GET, PUT, PATCH success
- `201` - POST creates resource
- `400` - Invalid input (missing fields, bad format)
- `401` - Auth failed (wrong password)
- `403` - Valid auth but insufficient role
- `404` - Resource not found
- `500` - Database/server error

---

## 3️⃣ YOUR API ENDPOINTS (5 mins)

| Endpoint | Method | Auth | Returns |
|----------|--------|------|---------|
| `/api/auth/login` | POST | ❌ | User object + role |
| `/api/auth/verify-password` | POST | ❌ | `{verified: true}` |
| `/api/users` | GET | ✓ Admin | `{users: [...]}` |
| `/api/users` | POST | ✓ Admin | `{user: {...}}` (201) |
| `/api/users/[id]` | PATCH | ✓ Admin | `{user: {...}}` |
| `/api/donors` | GET | ✓ | `{donors: [...]}` |
| `/api/donors` | POST | ✓ | `{donor: {...}}` (201) |
| `/api/donors/[id]` | GET/PUT/DELETE | ✓ | Donor object |
| `/api/payments` | GET/POST | ✓ | Payment object |
| `/api/expenditures` | GET/POST | ✓ | Expenditure object |
| `/api/settings` | GET/PUT | ✓ Admin | Settings object |
| `/api/dashboard/stats` | GET | ✓ | Dashboard stats |

---

## 4️⃣ AUTHENTICATION PATTERN (5 mins)

```typescript
// Login endpoint
export async function POST(req: NextRequest) {
  const { username, password } = await req.json()
  
  if (!username || !password) return json({error: "Required"}, 400)
  
  const user = await User.findOne({ username })
  if (!user || user.password !== password) {
    return json({error: "Invalid"}, 401)
  }
  
  return json({ user: { id, username, role } })
}

// Protected endpoint
export async function POST(req: NextRequest) {
  const role = req.headers.get("x-user-role")
  
  if (role !== "Admin") {
    return json({error: "Forbidden"}, 403)
  }
  
  // Admin-only logic
}
```

---

## 5️⃣ ERROR HANDLING TEMPLATE (5 mins)

```typescript
export async function POST(req: NextRequest) {
  try {
    await connectDB()
    
    const body = await req.json()
    
    // Validate input
    if (!body.name) {
      return NextResponse.json(
        { error: "Name required" }, 
        { status: 400 }
      )
    }
    
    // Check existence
    const exists = await Model.exists({ name: body.name })
    if (exists) {
      return NextResponse.json(
        { error: "Already exists" }, 
        { status: 400 }
      )
    }
    
    // Create
    const doc = await Model.create(body)
    
    return NextResponse.json(
      { data: doc }, 
      { status: 201 }
    )
    
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

## 6️⃣ KEY DIFFERENCES: Next.js Features (5 mins)

### Server vs Client Components
```typescript
// SERVER (default) - Can do:
export default function Page() {
  const data = await db.fetch()  // ✓
  const secret = process.env.SECRET  // ✓
  // ✗ Can't use hooks
}

// CLIENT - Can do:
"use client"
export function Component() {
  const [state, setState] = useState()  // ✓
  // ✗ Can't access DB
}
```

### Dynamic Rendering
```typescript
// Force always fresh (no caching)
export const dynamic = 'force-dynamic'

// Revalidate every 1 hour
export const revalidate = 3600

// Default - cache at build time
// (no export needed)
```

### Dynamic Params (IMPORTANT!)
```typescript
// ✓ CORRECT - Next.js 13.5+
type Params = { params: Promise<{ id: string }> }
export async function GET(req, { params }: Params) {
  const { id } = await params
}

// ✗ WRONG - Old pattern
export async function GET(req, { params }) {
  const { id } = params  // Not awaited!
}
```

---

## 7️⃣ COMMON CODE PATTERNS (5 mins)

### Input Validation
```typescript
const username = String(body.username || "").trim().toLowerCase()
if (!username) return json({error: "required"}, 400)
```

### Unique Values
```typescript
const exists = await User.exists({ username })
if (exists) return json({error: "taken"}, 400)
```

### Role Checking
```typescript
const allowedRoles = new Set(["Admin", "Trustee"])
if (!allowedRoles.has(role)) return json({error: "invalid"}, 400)
```

### Conditional Updates
```typescript
const update: Record<string, any> = {}
if (body.name !== undefined) update.name = body.name
if (body.email !== undefined) update.email = body.email
await User.findByIdAndUpdate(id, update)
```

### Safe Error Extraction
```typescript
catch (error) {
  const msg = error instanceof Error ? error.message : "Unknown"
  return json({error: msg}, 500)
}
```

---

## 8️⃣ INTERVIEW QUESTION QUICK ANSWERS (10 mins)

### Q: "Explain your architecture"
**A:** "Frontend React components → Next.js API routes → MongoDB database. Three layers, single codebase. Deployed on Vercel."

### Q: "How does authentication work?"
**A:** "User logs in with username/password. API validates against DB, returns user object with role. Client stores role and includes it in `x-user-role` header on future requests. API checks header to enforce authorization."

### Q: "Walk me through status codes"
**A:** "POST creates resource → 201. GET succeeds → 200. Bad input → 400. Wrong password → 401. Non-admin accessing admin route → 403. Resource missing → 404. Server error → 500."

### Q: "How do you validate data?"
**A:** "Three levels: (1) Type conversion & trim, (2) Required field checks, (3) Business logic validation (unique values, enum checks). Return 400 if any validation fails."

### Q: "What's the hardest part of this project?"
**A:** "Coordinating data across multiple modules—donors, payments, expenditures. Had to carefully design the schema relationships and handle complex queries for reports."

### Q: "What would you improve?"
**A:** 
- Add Bcrypt password hashing instead of plaintext
- Use JWT tokens instead of header-based auth
- Add rate limiting for brute force protection
- Implement proper error logging/monitoring
- Add comprehensive test coverage

---

## 9️⃣ QUICK MONGODB OPERATIONS (2 mins)

```typescript
// CRUD
const user = await User.create({ name, email })
const user = await User.findById(id)
const users = await User.find({}).lean()
await User.findByIdAndUpdate(id, { name: "New" })
await User.findByIdAndDelete(id)

// Checks
const exists = await User.exists({ username })
const count = await User.countDocuments()

// Queries
const user = await User.findOne({ username, status: "Active" })
const users = await User.find({ role: "Admin" }).sort({ name: 1 })
```

---

## 🔟 THE 2-MINUTE VERSION

If you have only 2 minutes before interview:

**Know this:**
1. You use Next.js App Router (not Pages)
2. API routes match HTTP methods (GET, POST, etc.)
3. Status codes: 200 OK, 201 Created, 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 500 Server Error
4. Auth: Login validates password, returns user with role, role checked on protected routes
5. Database: MongoDB + Mongoose, connect before queries
6. Deployment: Vercel (just `vercel deploy`)

**Practice saying:**
> "I built a full-stack dashboard with Next.js API routes, MongoDB database, and deployed on Vercel. Authentication is role-based via headers. I return appropriate HTTP status codes for each scenario."

---

## 📋 REAL CODE FROM YOUR PROJECT

### Actual login endpoint
```typescript
// src/app/api/auth/login/route.ts
const user = await User.findOne({ username, status: "Active" })
if (!user || user.password !== password) {
  return NextResponse.json({ error: "Invalid username or password" }, { status: 401 })
}

user.lastLogin = new Date()
await user.save()

return NextResponse.json({
  user: {
    id: String(user._id),
    username: user.username,
    name: user.name,
    role: user.role,
  },
})
```

### Actual admin check
```typescript
// src/app/api/users/route.ts
function assertAdmin(req: NextRequest) {
  return req.headers.get("x-user-role") === "Admin"
}

export async function POST(req: NextRequest) {
  if (!assertAdmin(req)) {
    return NextResponse.json({ error: "Forbidden" }, { status: 403 })
  }
  
  // Create user...
  return NextResponse.json({ user: userObject }, { status: 201 })
}
```

### Actual validation pattern
```typescript
// src/app/api/users/route.ts
const password = String(body.password || "").trim()
const role = String(body.role || "Viewer")

if (!password) {
  return NextResponse.json({ error: "Missing required fields" }, { status: 400 })
}

const allowedRoles = new Set(["Admin", "Trustee", "Auditor", "Viewer"])
if (!allowedRoles.has(role)) {
  return NextResponse.json({ error: "Invalid role" }, { status: 400 })
}
```

---

## ✅ FINAL CHECKLIST

Before you go into interview:

- [ ] You can draw your architecture (Frontend → API → DB)
- [ ] You know all 7 HTTP status codes and when to use them
- [ ] You understand the login + auth header flow
- [ ] You can explain one of your API endpoints from memory
- [ ] You know why you use Next.js over plain Node.js
- [ ] You can discuss 2-3 security improvements
- [ ] You have 2-3 questions ready about their tech
- [ ] You know how to deploy to Vercel
- [ ] You understand the difference between server and client components

---

**You got this! 💪 Go crush that interview!**

*Review time: 20-30 minutes*
*Keep this open on your phone during breaks!*
