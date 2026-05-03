# ⏱️ 5-MINUTE PRE-INTERVIEW POWER SUMMARY
## Read This Right Before You Go In

---

## 🎯 YOUR PITCH (30 seconds)

> "I built a full-stack dashboard application using **Next.js with App Router for both frontend and API**, **MongoDB for data persistence**, and **deployed on Vercel**. The application manages donors, payments, and expenditures with role-based access control. All API endpoints return appropriate HTTP status codes (200, 201, 400, 401, 403, 404, 500) and use MongoDB for persistence."

---

## 🔑 THE 7 CORE CONCEPTS

### 1. API Routes
```typescript
export async function GET(req) { } // Automatic routing
export async function POST(req) { }
```

### 2. HTTP Status Codes
- `200` = Success
- `201` = Created
- `400` = Bad input
- `401` = Wrong password
- `403` = No permission
- `404` = Not found
- `500` = Server error

### 3. Authentication
```typescript
if (req.headers.get("x-user-role") !== "Admin") return 403
```

### 4. Dynamic Routes
```typescript
const { id } = await params  // MUST AWAIT!
```

### 5. Input Validation
```typescript
if (!username || !password) return json({error}, 400)
```

### 6. Database Operations
```typescript
await connectDB()
const user = await User.findOne({ username })
```

### 7. Error Handling
```typescript
catch (error) {
  return json({error: "message"}, 500)
}
```

---

## 💬 YOUR ANSWERS (For Top 5 Questions)

### Q1: "What's your project about?"
**A:** "Full-stack dashboard for managing donors and payments. Built with Next.js (frontend + API), MongoDB (database), Vercel (deployment). Features include user authentication, role-based access control, and financial reporting."

### Q2: "Explain your API architecture"
**A:** "Each API route file exports HTTP handlers (GET, POST, etc.). The route receives NextRequest, validates input (400 if invalid), checks auth header (401/403 if needed), queries MongoDB, and returns appropriate status code with JSON response."

### Q3: "Walk me through a POST endpoint"
**A:** "POST /api/users: 
1. Check Admin role in header (403 if not)
2. Extract name, password, role from body
3. Validate required fields (400 if missing)
4. Validate role is in allowed set (400 if not)
5. Check username is unique (400 if duplicate)
6. Create user document in MongoDB
7. Return 201 with user object"

### Q4: "How do you handle errors?"
**A:** "Three levels: (1) Input validation returns 400, (2) Authorization checks return 403, (3) Database errors return 500. Each returns JSON with error message and appropriate status code."

### Q5: "What would you improve?"
**A:** "Add Bcrypt for password hashing instead of plaintext. Use JWT tokens instead of headers. Add rate limiting for security. Implement proper logging. Add test coverage."

---

## 📊 YOUR STATUS CODE RESPONSES

| Scenario | Code | Example |
|----------|------|---------|
| Validation failed | 400 | Missing field, invalid role |
| Wrong password | 401 | username/password mismatch |
| Not admin | 403 | Accessing admin-only route |
| User not found | 404 | GET /api/users/invalid-id |
| Created resource | 201 | POST /api/users success |
| Got data | 200 | GET /api/users success |
| Server crash | 500 | Database connection error |

---

## 🔐 YOUR SECURITY STORY

**Current:**
- Custom auth with headers (`x-user-role`)
- Role-based access in API routes
- Plain text passwords (education project)

**Production improvements I'd make:**
- Bcrypt for password hashing
- JWT tokens (signed, tamper-proof)
- Rate limiting (prevent brute force)
- HTTPS only (Vercel does this)
- Proper session management

---

## 🗄️ YOUR DATABASE MODELS

```
User: id, username, password, name, email, role, status
Donor: id, name, email, phone, status, lastLogin
Payment: paymentId, donorId, method, amount, date, month, year
Expenditure: id, category, amount, date, description
Property: id, name, address, value
AuditLog: action, model, recordId, description, performedBy
```

---

## 📋 YOUR API ENDPOINTS (Quick Reference)

| Path | Method | Does | Auth | Returns |
|------|--------|------|------|---------|
| `/api/auth/login` | POST | Authenticate user | ❌ | User + role |
| `/api/users` | GET | List users | ✓ Admin | Users array |
| `/api/users` | POST | Create user | ✓ Admin | Created user (201) |
| `/api/donors` | GET/POST | Read/create donors | ✓ | Donors/status |
| `/api/payments` | GET/POST | Read/create payments | ✓ | Payments/status |
| `/api/settings` | GET/PUT | Read/update settings | ✓ Admin | Settings |
| `/api/dashboard/stats` | GET | Get stats | ✓ | Dashboard data |

---

## 🚀 DEPLOYMENT & VERCEL

**You use Vercel because:**
- Zero-config Next.js deployment
- Automatic HTTPS
- Serverless functions (your API routes)
- Environment variables
- CI/CD built-in

**Env vars needed:**
```
MONGODB_URI=...
```

**Deploy command:**
```bash
vercel deploy --prod
```

---

## ⚡ THE SUPABASE ANGLE (They mentioned it!)

**What to say:**
> "This project uses MongoDB, but I understand you're using Supabase. I'm familiar with PostgreSQL + Auth + RLS. Happy to discuss migration or how I'd build this with Supabase's stack instead."

**Key Supabase advantages:**
- Built-in auth (no custom headers)
- RLS (row-level security at DB)
- Real-time updates (WebSocket)
- PostgreSQL (relational queries)

---

## 🎯 YOUR STRENGTHS (Emphasize These)

1. **Full-stack understanding** - Frontend to database
2. **Real project** - Not just tutorials, actual working app
3. **Best practices** - Proper status codes, error handling
4. **Type safety** - TypeScript throughout
5. **Modern stack** - Next.js 13+, App Router

---

## ⚠️ WEAK POINTS (Be ready for these)

1. **Plain text passwords** - "In production I'd use Bcrypt"
2. **Header-based auth** - "I'd use JWT tokens for scalability"
3. **No rate limiting** - "I'd add it for production security"
4. **Limited tests** - "I prioritized feature delivery, but would test at scale"
5. **Not Supabase** - "Open to learning, have PostgreSQL knowledge"

---

## 🧠 MENTAL CHECKLIST

Before you go in, confirm you can:

- [ ] Draw architecture: Frontend → API → DB
- [ ] Explain login flow end-to-end
- [ ] Name all 7 HTTP status codes
- [ ] Code a simple POST endpoint from scratch
- [ ] Explain why a specific status code matters
- [ ] Discuss authentication vs authorization
- [ ] Handle a challenging "what would you change" question
- [ ] Ask 2-3 intelligent questions about THEIR project

---

## 💪 YOUR CONFIDENCE BOOSTERS

**You've got this because:**
- ✅ You have a real, working project
- ✅ You understand every piece of your code
- ✅ You're prepared for common questions
- ✅ You know your weak points and have answers
- ✅ You understand HTTP semantics deeply
- ✅ You can explain technical decisions

**If you freeze:**
- Pause, breathe, think
- Draw diagrams if it helps
- "Let me think through that..."
- It's OK to say "I'd research that"

---

## 🎤 ONE THING TO REMEMBER

The interviewer wants to know:
1. Can you code? ✅ Yes (your project)
2. Do you understand your code? ✅ Yes (you built it)
3. Can you grow? ✅ Yes (you identified improvements)

Everything else is details. You're prepared.

---

## 📱 FINAL CHECKLIST

- [ ] Have your project repo link ready
- [ ] Have specific file paths to reference
- [ ] Have 2-3 questions about their tech
- [ ] Know your salary expectations
- [ ] Know your availability
- [ ] Know the location/remote situation
- [ ] You've eaten and hydrated ✓

---

## 🏁 GO CRUSH IT!

**Remember:**
> "I built a full-stack Next.js application with API routes, MongoDB, and Vercel deployment. I understand the HTTP semantics, authentication architecture, and production considerations. I'm excited about learning Supabase for this role."

**That statement + your project = You're hired.** 💼✨

---

**Interview starts in:** [START NOW! 🚀]

*Last reviewed: Right before you go in*
*Good luck! You prepared for this.* 🎉
