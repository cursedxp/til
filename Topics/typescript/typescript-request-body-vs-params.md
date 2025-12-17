# TypeScript: Request Body vs URL Params in Next.js

**Date**: 2025-10-17
**Topic**: Understanding the fundamental difference between request body data and URL parameters

---

## 🎯 The Core Confusion

When working with Next.js API routes, beginners often confuse:
- **Request Body**: Data sent in POST/PUT/PATCH requests
- **URL Params**: Dynamic segments extracted from the URL path

These are **completely different concepts** and come from **different sources**!

---

## 📚 Two Different Data Sources

### **1. Request Body**

**What it is**: Data sent in the HTTP request body
**Used for**: POST, PUT, PATCH requests
**Format**: Usually JSON
**How to access**: `await request.json()`

```typescript
// Client sends:
POST /api/tasks
Content-Type: application/json

{
  "title": "Buy milk",
  "description": "Get 2 gallons",
  "completed": false
}

// Server receives:
export async function POST(request: NextRequest) {
  const body = await request.json();
  console.log(body.title); // "Buy milk"
}
```

---

### **2. URL Parameters (params)**

**What it is**: Dynamic segments in the URL path
**Used for**: GET, PUT, DELETE requests (identifying resources)
**Format**: Part of the URL structure
**How to access**: Second parameter `{ params }`

```typescript
// Client sends:
GET /api/tasks/123

// Server receives:
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  console.log(params.id); // "123"
}
```

---

## 🔍 Visual Comparison

### **Request Body Example**

```
Client Request
    ↓
POST /api/tasks
Body: { "title": "Buy milk", "completed": false }
    ↓
┌─────────────────────────────────────┐
│ API Route: app/api/tasks/route.ts  │
│                                     │
│ export async function POST(        │
│   request: NextRequest              │
│ ) {                                 │
│   const body = await request.json()│
│   //          ^^^^^^^^^^^^^^^^^^^^ │
│   //          From HTTP body       │
│                                     │
│   console.log(body.title)          │
│   // "Buy milk"                    │
│ }                                   │
└─────────────────────────────────────┘
```

---

### **URL Params Example**

```
Client Request
    ↓
GET /api/tasks/123
             ^^^
             Param value
    ↓
┌──────────────────────────────────────────┐
│ API Route: app/api/tasks/[id]/route.ts  │
│                          ^^^^            │
│                          Param name      │
│                                          │
│ export async function GET(              │
│   request: NextRequest,                 │
│   { params }: { params: { id: string } }│
│   //^^^^^^                   ^^         │
│   //From Next.js routing     Param name │
│ ) {                                      │
│   console.log(params.id)                │
│   // "123"                              │
│ }                                        │
└──────────────────────────────────────────┘
```

---

## ❌ The Common Mistake

### **Wrong: Treating Params Like Request Body**

```typescript
// ❌ WRONG - This is mixing two concepts!
export async function POST(
  request: NextRequest,
  {
    params,
  }: {
    params: {
      title: string;      // ❌ This is NOT params!
      description: string; // ❌ This is body data!
      completed: boolean;  // ❌ Doesn't belong here!
    };
  }
) {
  // This won't work!
  // params is for URL segments, not request body!
}
```

**Why it's wrong:**
1. `params` only contains dynamic URL segments like `[id]`
2. Request body data comes from `await request.json()`
3. Static routes like `/api/tasks` don't have params at all!

---

### **Right: Using Each Correctly**

```typescript
// ✅ CORRECT - Static route with request body
export async function POST(request: NextRequest) {
  const body = await request.json();

  const { title, description, completed } = body;
  // ✅ Body data from request.json()
}

// ✅ CORRECT - Dynamic route with params
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const taskId = params.id;
  // ✅ ID from URL segment
}
```

---

## 🎓 When to Use Which

### **Use Request Body When:**

- ✅ Creating new resources (POST)
- ✅ Updating existing resources (PUT/PATCH)
- ✅ Sending complex data structures
- ✅ Sending sensitive data (not in URL)
- ✅ Sending large amounts of data

**Example**:
```typescript
// POST /api/tasks
// Body: { "title": "New task", "description": "Details..." }

export async function POST(request: NextRequest) {
  const body = await request.json();
  const newTask = await createTask(body);
  return NextResponse.json(newTask, { status: 201 });
}
```

---

### **Use URL Params When:**

- ✅ Identifying a specific resource (GET by ID)
- ✅ Updating a specific resource (PUT /tasks/123)
- ✅ Deleting a specific resource (DELETE /tasks/123)
- ✅ Dynamic routing (e.g., `/users/[userId]/posts/[postId]`)

**Example**:
```typescript
// GET /api/tasks/123
// params.id = "123"

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const task = await getTask(params.id);
  return NextResponse.json(task);
}
```

---

## 📊 Side-by-Side Comparison

| Aspect | Request Body | URL Params |
|--------|-------------|------------|
| **Source** | HTTP request body | URL path segments |
| **Access** | `await request.json()` | `{ params }` second parameter |
| **Used in** | POST, PUT, PATCH | All methods (GET, POST, PUT, DELETE) |
| **Route type** | Static or dynamic | Only dynamic routes `[id]` |
| **Visibility** | Hidden in HTTP body | Visible in URL |
| **Size limit** | Large (MB) | Small (URL length limits) |
| **Format** | JSON, FormData, etc. | Always strings |
| **Example** | `{ "title": "Task" }` | `/api/tasks/123` → `params.id` |

---

## 💡 Combining Both

Often, you need **both** params and body in the same request:

### **Example: Update Specific Task**

```typescript
// PUT /api/tasks/123
// Body: { "title": "Updated title", "completed": true }

export async function PUT(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  // ✅ Get ID from URL params
  const taskId = params.id; // "123"

  // ✅ Get update data from request body
  const body = await request.json();
  const { title, completed } = body;

  // Update the specific task
  const updated = await updateTask(taskId, { title, completed });

  return NextResponse.json(updated);
}
```

**Why both?**
- **params.id**: Tells us **which** task to update
- **body**: Tells us **what** to update

---

## 🛠️ Real-World Examples

### **Example 1: Create Task (Body Only)**

```typescript
// File: app/api/tasks/route.ts
// POST /api/tasks
// Body: { "title": "New task", "description": "Details" }

export async function POST(request: NextRequest) {
  const body = await request.json();

  const newTask = {
    title: body.title,
    description: body.description,
    completed: false
  };

  const created = await db.tasks.create({ data: newTask });
  return NextResponse.json(created, { status: 201 });
}
```

---

### **Example 2: Get Task (Params Only)**

```typescript
// File: app/api/tasks/[id]/route.ts
// GET /api/tasks/123

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const task = await db.tasks.findUnique({
    where: { id: params.id }
  });

  if (!task) {
    return NextResponse.json(
      { error: "Task not found" },
      { status: 404 }
    );
  }

  return NextResponse.json(task);
}
```

---

### **Example 3: Update Task (Both!)**

```typescript
// File: app/api/tasks/[id]/route.ts
// PUT /api/tasks/123
// Body: { "title": "Updated", "completed": true }

export async function PUT(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  // Params: Which task?
  const taskId = params.id;

  // Body: What to update?
  const body = await request.json();

  const updated = await db.tasks.update({
    where: { id: taskId },
    data: {
      title: body.title,
      completed: body.completed
    }
  });

  return NextResponse.json(updated);
}
```

---

## 🎯 Decision Tree

```
What data do I need to access?
    │
    ├─ Data identifying WHICH resource?
    │  (e.g., task ID, user ID)
    │  └─> Use URL PARAMS
    │      Example: /api/tasks/[id]
    │      Access: params.id
    │
    └─ Data describing WHAT to create/update?
       (e.g., task title, description, status)
       └─> Use REQUEST BODY
           Example: { "title": "..." }
           Access: await request.json()
```

---

## ⚠️ Common Pitfalls

### **Pitfall 1: Using Body for Resource IDs**

```typescript
// ❌ Bad - ID should be in URL
POST /api/tasks
Body: { "id": "123", "title": "Updated" }

// ✅ Good - ID in URL, data in body
PUT /api/tasks/123
Body: { "title": "Updated" }
```

---

### **Pitfall 2: Putting Sensitive Data in URL**

```typescript
// ❌ Bad - password visible in URL!
GET /api/login?username=john&password=secret123

// ✅ Good - credentials in body
POST /api/login
Body: { "username": "john", "password": "secret123" }
```

---

### **Pitfall 3: Using Params for Static Routes**

```typescript
// ❌ Bad - no params in static route!
// File: app/api/tasks/route.ts
export async function POST(
  request: NextRequest,
  { params }: { params: { title: string } } // ❌ Won't work!
) { }

// ✅ Good - only request parameter
export async function POST(request: NextRequest) {
  const body = await request.json();
  const { title } = body;
}
```

---

## 📋 Quick Reference

### **When You See This URL:**

| URL Pattern | Has Params? | Has Body? |
|-------------|-------------|-----------|
| `POST /api/tasks` | ❌ No | ✅ Yes |
| `GET /api/tasks` | ❌ No | ❌ No |
| `GET /api/tasks/123` | ✅ Yes (`id`) | ❌ No |
| `PUT /api/tasks/123` | ✅ Yes (`id`) | ✅ Yes |
| `DELETE /api/tasks/123` | ✅ Yes (`id`) | ❌ No |

---

## 🎯 Key Takeaways

1. ✅ **Request body** = Data in HTTP body (`await request.json()`)
2. ✅ **URL params** = Dynamic URL segments (`params.id`)
3. ✅ **Never confuse them** - they're from different sources
4. ✅ **Static routes** (`/api/tasks`) = No params
5. ✅ **Dynamic routes** (`/api/tasks/[id]`) = Has params
6. ✅ **Can use both** in the same request (e.g., PUT)
7. ✅ **Body for WHAT**, **params for WHICH**

---

## 📝 Practice Exercise

Identify what's wrong and fix it:

```typescript
// File: app/api/tasks/route.ts
// ❌ What's wrong with this?
export async function POST(
  request: NextRequest,
  { params }: { params: { title: string; description: string } }
) {
  const task = await createTask(params.title, params.description);
  return NextResponse.json(task);
}
```

<details>
<summary>Click for solution</summary>

**Problems:**
1. Static route `/api/tasks` doesn't have params
2. Task data should come from request body, not params

**Fixed version:**
```typescript
// File: app/api/tasks/route.ts
// ✅ Correct
export async function POST(request: NextRequest) {
  const body = await request.json();

  const task = await createTask({
    title: body.title,
    description: body.description
  });

  return NextResponse.json(task, { status: 201 });
}
```

</details>

---

_Understanding the difference between request body and URL params is fundamental to building RESTful APIs._
