# Next.js API Routes & HTTP Methods

## What I Learned

This document summarizes my understanding of Next.js API Routes and HTTP methods.

---

## 📁 API Routes Structure

- Files in `app/api/` automatically become API endpoints
- `app/api/tasks/route.ts` → accessible at `/api/tasks`
- `app/api/users/[id]/route.ts` → accessible at `/api/users/123` (dynamic route)

---

## 📨 Request & Response Objects

### Request Object
Contains everything the client sent to the API:

```typescript
export async function POST(request: Request) {
  const body = await request.json()           // Data from request body
  const searchParams = request.nextUrl.searchParams  // Query params (?name=John)
  const headers = request.headers             // Headers (auth tokens, etc)
  const url = request.url                     // Full URL
}
```

### Response Object
`Response.json()` creates a complete HTTP Response with:

1. **Body** - Your data converted to JSON string
2. **Headers** - Automatically adds `Content-Type: application/json`
3. **Status code** - Default 200 (or custom)

```typescript
return Response.json(
  { tasks: [...] },    // Body
  {
    status: 200,       // Status code
    headers: {         // Optional additional headers
      'X-Custom-Header': 'value'
    }
  }
)
```

---

## 🔄 HTTP Methods Explained

### GET - View/Read Data
- **Purpose**: Retrieve data (NO database changes)
- **Has body?**: NO (data comes from URL/query params)
- **Use case**: Viewing lists, getting single items

```typescript
// GET /api/tasks
export async function GET(request: Request) {
  const tasks = await db.getAllTasks()
  return Response.json({ tasks })
}
```

### POST - Create New Data
- **Purpose**: Create NEW resource in database
- **Has body?**: YES (new data in request body)
- **Use case**: Creating new records

```typescript
// POST /api/tasks
export async function POST(request: Request) {
  const { title, completed } = await request.json()
  const newTask = await db.createTask({ title, completed })
  return Response.json({ task: newTask }, { status: 201 })
}
```

### PATCH - Update Partially (SAFE) ✅
- **Purpose**: Update ONLY specific fields
- **Has body?**: YES (only fields to change)
- **Use case**: Most updates (mark complete, change title, etc.)
- **Safe**: Fields not included remain unchanged

```typescript
// PATCH /api/tasks/123
export async function PATCH(
  request: Request,
  { params }: { params: { id: string } }
) {
  const updates = await request.json()  // Only { completed: true }
  const updated = await db.updateTask(params.id, updates)
  return Response.json({ task: updated })
}
```

### PUT - Replace Completely (DANGEROUS) ⚠️
- **Purpose**: Replace ENTIRE resource
- **Has body?**: YES (must include ALL fields)
- **Use case**: Full replacement, imports
- **Dangerous**: Missing fields get removed/set to null

```typescript
// PUT /api/tasks/123
export async function PUT(
  request: Request,
  { params }: { params: { id: string } }
) {
  const newData = await request.json()  // Must include ALL fields
  const updated = await db.replaceTask(params.id, newData)
  return Response.json({ task: updated })
}
```

### DELETE - Remove Data
- **Purpose**: Delete a resource
- **Has body?**: Usually NO
- **Use case**: Removing records

```typescript
// DELETE /api/tasks/123
export async function DELETE(
  request: Request,
  { params }: { params: { id: string } }
) {
  await db.deleteTask(params.id)
  return Response.json({ success: true }, { status: 204 })
}
```

### HEAD & OPTIONS
- **HEAD**: Same as GET but only returns headers (no body)
- **OPTIONS**: Returns allowed methods (CORS preflight)
- Rarely used in typical applications

---

## 🎯 Key Differences

### "Has Body?" Explained
- **Body** = The main data payload in HTTP request
- **GET**: Data in URL (`?userId=123`) - NO body
- **POST/PUT/PATCH/DELETE**: Data in request body - HAS body

### Database Response vs HTTP Response
```typescript
// Database returns plain JavaScript object/array:
const tasks = await db.query('SELECT * FROM tasks')
// tasks = [{ id: 1, title: 'Buy milk' }]

// Response.json() converts to HTTP Response:
return Response.json({ tasks })
// Creates Response with body, headers, and status code
```

### PUT vs PATCH - The Critical Difference

| Method | What Happens | Safety |
|--------|--------------|--------|
| **PUT** | Replaces ENTIRE resource | ⚠️ **Dangerous** - Missing fields get lost |
| **PATCH** | Updates ONLY sent fields | ✅ **Safe** - Other fields remain unchanged |

**Example:**
```typescript
// Database: { id: 1, title: 'Buy milk', completed: false, notes: 'Organic' }

// PUT (forgot 'notes'):
{ title: 'Buy milk', completed: true }
// Result: { id: 1, title: 'Buy milk', completed: true, notes: null } ❌ Lost notes!

// PATCH (only what changed):
{ completed: true }
// Result: { id: 1, title: 'Buy milk', completed: true, notes: 'Organic' } ✅ Safe!
```

---

## 📊 Quick Decision Guide

| User Action | HTTP Method |
|-------------|-------------|
| View list of tasks | `GET /api/tasks` |
| View single task | `GET /api/tasks/123` |
| Create new task | `POST /api/tasks` |
| Mark task complete | `PATCH /api/tasks/123` |
| Edit task title | `PATCH /api/tasks/123` |
| Replace entire task | `PUT /api/tasks/123` |
| Delete task | `DELETE /api/tasks/123` |

---

## 💡 Key Takeaways

1. **GET** = View data (no changes, no body)
2. **POST** = Create new data (has body)
3. **PATCH** = Safe updates (only send what changed) ← **Use this 99% of the time**
4. **PUT** = Dangerous full replacement (must send everything)
5. **DELETE** = Remove data
6. **Response.json()** packages body + headers + status into HTTP response
7. Database returns plain JavaScript, Response.json() converts to HTTP format

---

_Date: October 14, 2025_
