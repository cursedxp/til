# Next.js Route Handler Parameters

**Date**: 2025-10-17
**Topic**: Understanding the two parameters Next.js passes to route handlers

---

## 🎯 The Two Parameters

**Next.js ALWAYS passes exactly 2 parameters to route handlers:**

```typescript
export async function GET(
  parameter1,  // HTTP request (source of truth)
  parameter2   // Route context (helper from Next.js)
) { }
```

---

## 📚 Parameter 1: `request` (HTTP Request)

### **The Source of Truth**

```typescript
export async function GET(request: NextRequest) {
  //                       ^^^^^^^^ variable name
  //                                ^^^^^^^^^^^ type
}
```

### **What `NextRequest` Contains**:

```typescript
// URL information
request.url           // "http://localhost:3000/api/tasks"
request.nextUrl       // Next.js enhanced URL object

// HTTP method
request.method        // "GET", "POST", etc.

// Headers
request.headers.get("authorization")
request.headers.get("content-type")

// Cookies (Next.js specific!)
request.cookies.get("token")
request.cookies.get("session")

// Query parameters (manual extraction)
const { searchParams } = new URL(request.url);
searchParams.get("page")     // ?page=1
searchParams.get("status")   // ?status=completed

// Body (for POST/PUT/PATCH)
const body = await request.json()
```

---

## 📚 Parameter 2: `context` (Route Context)

### **The Convenience Helper**

```typescript
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
  //^^^^^^ value (destructured)
  //          ^^^^^^^^^^^^^^^^^^^^^^^^^ type
) {
  const id = params.id; // Easy access!
}
```

### **What Context Contains**:

```typescript
// context object structure:
{
  params: {
    // Dynamic route segments
    id: "123",
    // If you have [userId]/posts/[postId]
    userId: "5",
    postId: "42"
  }
}
```

---

## 🎓 Request vs Context: The Key Insight

> **`request` has EVERYTHING (source of truth).**
>
> **`context` is a CONVENIENCE HELPER from Next.js to extract useful routing info.**

### **Visual Breakdown**:

```
User Request: GET /api/tasks/123
              ↓
┌─────────────────────────────────────────────┐
│ Next.js creates TWO SEPARATE objects:      │
│                                             │
│ 1. request (HTTP info)                     │
│    {                                       │
│      url: "/api/tasks/123",                │
│      method: "GET",                        │
│      headers: {...},                       │
│      cookies: {...}                        │
│    }                                       │
│                                             │
│ 2. context (routing info)                  │
│    {                                       │
│      params: { id: "123" }                 │
│    }                                       │
└─────────────────────────────────────────────┘
              ↓
Next.js calls: GET(request, context)
```

**Key Point**: These are **TWO SEPARATE objects**, NOT nested!

```typescript
// ❌ Wrong understanding:
request → contains context → contains params

// ✅ Correct understanding:
Next.js passes TWO separate things:
  1. request (HTTP info)
  2. context (route info with params)
```

---

## 💡 Why Next.js Provides Context

### **Without Context** (Manual Extraction):

```typescript
export async function GET(request: NextRequest) {
  // ❌ Annoying manual extraction:
  const url = new URL(request.url);
  const pathParts = url.pathname.split('/');
  const id = pathParts[pathParts.length - 1]; // Extract "123"

  // Lots of error-prone string manipulation! 😫
}
```

### **With Context** (Next.js Helper):

```typescript
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  // ✅ Clean and easy:
  const id = params.id; // "123"

  // Beautiful! 😊
}
```

---

## 🛠️ Dynamic Route Segments

### **How Next.js Extracts Params**:

When you create a dynamic route with `[id]`:

```
app/api/tasks/[id]/route.ts
              ^^^^
              This folder name tells Next.js!
```

Next.js automatically:
1. **Extracts** the value from URL
2. **Creates** the context object: `{ params: { id: "123" } }`
3. **Passes** it as the second parameter

### **Flow**:

```
File: app/api/tasks/[id]/route.ts
                     ^^^
         ↓
URL: /api/tasks/123
                ^^^
         ↓
Next.js extracts: "123"
         ↓
Next.js creates: { params: { id: "123" } }
         ↓
Next.js calls: GET(request, { params: { id: "123" } })
                                       ↑
                      You access it: params.id
```

---

## 🎯 Multiple Dynamic Segments

### **Nested Dynamic Routes**:

```
app/api/users/[userId]/posts/[postId]/route.ts
              ^^^^^^^^       ^^^^^^^^
```

### **What Next.js Passes**:

```typescript
export async function GET(
  request: NextRequest,
  { params }: { params: { userId: string; postId: string } }
) {
  const userId = params.userId;   // "5"
  const postId = params.postId;   // "42"
}

// URL: /api/users/5/posts/42
// params = { userId: "5", postId: "42" }
```

---

## 📊 Complete Example

### **File**: `app/api/tasks/[id]/route.ts`

```typescript
import { NextRequest, NextResponse } from "next/server";
import { getTask, updateTask, deleteTask } from "@/lib/tasks";

// GET single task
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  // Access dynamic route param
  const taskId = params.id;

  // Access query params (if needed)
  const { searchParams } = new URL(request.url);
  const include = searchParams.get("include"); // ?include=comments

  // Access headers
  const auth = request.headers.get("authorization");

  const task = await getTask(taskId);

  if (!task) {
    return NextResponse.json(
      { error: "Task not found" },
      { status: 404 }
    );
  }

  return NextResponse.json(task);
}

// PUT update task
export async function PUT(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const taskId = params.id;
  const body = await request.json(); // Access request body

  const updated = await updateTask(taskId, body);
  return NextResponse.json(updated);
}

// DELETE task
export async function DELETE(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const taskId = params.id;
  await deleteTask(taskId);

  return NextResponse.json({ success: true });
}
```

---

## 🎓 TypeScript Type Annotations

### **Understanding the Types**:

```typescript
export async function GET(
  request: NextRequest,
  //^^^^^^ value (the actual request object)
  //       ^^^^^^^^^^^ type (NextRequest class)

  { params }: { params: { id: string } }
  // ^^^^^^ value (destructured params)
  //          ^^^^^^^^^^^^^^^^^^^^^^^^^ type (object shape)

): Promise<NextResponse> {
// ^^^^^^^^^^^^^^^^^^^^^ return type

  const id: string = params.id;
  //    ^^ value
  //       ^^^^^^ type

  return NextResponse.json({ id });
}
```

### **The Pattern**:
```typescript
variableName: TypeName
     ↑            ↑
   VALUE        TYPE
```

---

## 💡 Key Takeaways

1. ✅ Next.js passes **TWO separate parameters**: `request` and `context`
2. ✅ `request` = Source of truth (HTTP info, headers, body, cookies)
3. ✅ `context` = Helper (extracted route params)
4. ✅ Dynamic segments like `[id]` are automatically extracted
5. ✅ Context is NOT nested inside request
6. ✅ You can access both in every route handler

---

## 📝 Practice Questions

### **Question 1**: What does `params` contain for this URL?
```typescript
// File: app/api/users/[userId]/tasks/[taskId]/route.ts
// URL: /api/users/42/tasks/123
```

### **Question 2**: How do you access query parameters?
```typescript
// URL: /api/tasks?status=completed&page=2
```

### **Question 3**: Where do cookies come from?
- A) `params.cookies`
- B) `request.cookies`
- C) `context.cookies`

---

_Understanding the two parameters Next.js passes is essential for building robust API routes._
