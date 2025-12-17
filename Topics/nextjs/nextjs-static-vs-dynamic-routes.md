# Next.js: Static vs Dynamic API Routes

**Date**: 2025-10-17
**Topic**: Understanding when to use static routes vs dynamic routes and how they affect TypeScript typing

---

## 🎯 The Two Types of Routes

In Next.js, there are two fundamental types of API routes:

1. **Static Routes**: Fixed paths like `/api/tasks`
2. **Dynamic Routes**: Variable paths like `/api/tasks/[id]`

Each type has **different function signatures** and **different use cases**.

---

## 📚 Static Routes

### **What They Are**

Routes with a **fixed, unchanging path** that don't include variable segments.

### **File Structure**

```
app/
  api/
    tasks/
      route.ts          ← Static route
    users/
      route.ts          ← Static route
```

### **URL Examples**

```
/api/tasks              ← Matches tasks/route.ts
/api/users              ← Matches users/route.ts
/api/products           ← Matches products/route.ts
```

---

### **Function Signature**

Static routes **only need ONE parameter**: `request`

```typescript
export async function GET(request: NextRequest) {
  //                       ^^^^^^^^^^^^^^^^^^^
  //                       ONLY parameter needed
}

export async function POST(request: NextRequest) {
  //                        ^^^^^^^^^^^^^^^^^^^
  //                        ONLY parameter needed
}
```

**Why?** No dynamic segments means no `params` to extract!

---

### **Complete Example**

```typescript
// File: app/api/tasks/route.ts
import { NextRequest, NextResponse } from "next/server";
import { getTasks, createTask } from "@/lib/tasks";

// GET /api/tasks - Get all tasks
export async function GET(request: NextRequest) {
  const tasks = await getTasks();
  return NextResponse.json(tasks);
}

// POST /api/tasks - Create new task
export async function POST(request: NextRequest) {
  const body = await request.json();

  const newTask = {
    title: body.title,
    description: body.description,
    completed: false
  };

  const created = await createTask(newTask);
  return NextResponse.json(created, { status: 201 });
}
```

---

## 📚 Dynamic Routes

### **What They Are**

Routes with **variable segments** in the path, denoted by `[brackets]`.

### **File Structure**

```
app/
  api/
    tasks/
      [id]/
        route.ts        ← Dynamic route (has [id])
    users/
      [userId]/
        posts/
          [postId]/
            route.ts    ← Dynamic route (has [userId] and [postId])
```

### **URL Examples**

```
/api/tasks/123          ← Matches tasks/[id]/route.ts (id = "123")
/api/tasks/abc-xyz      ← Matches tasks/[id]/route.ts (id = "abc-xyz")
/api/users/5/posts/42   ← Matches users/[userId]/posts/[postId]/route.ts
```

---

### **Function Signature**

Dynamic routes need **TWO parameters**: `request` and `{ params }`

```typescript
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
  //^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  //SECOND parameter for dynamic segments
) {
  const id = params.id; // Access the dynamic segment
}
```

**Why?** Next.js extracts dynamic segments and provides them as `params`!

---

### **Complete Example**

```typescript
// File: app/api/tasks/[id]/route.ts
import { NextRequest, NextResponse } from "next/server";
import { getTask, updateTask, deleteTask } from "@/lib/tasks";

// GET /api/tasks/123 - Get single task
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const task = await getTask(params.id);

  if (!task) {
    return NextResponse.json(
      { error: "Task not found" },
      { status: 404 }
    );
  }

  return NextResponse.json(task);
}

// PUT /api/tasks/123 - Update task
export async function PUT(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const body = await request.json();
  const updated = await updateTask(params.id, body);
  return NextResponse.json(updated);
}

// DELETE /api/tasks/123 - Delete task
export async function DELETE(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  await deleteTask(params.id);
  return NextResponse.json({ success: true });
}
```

---

## 🔍 Visual Comparison

### **Static Route Flow**

```
Client Request: GET /api/tasks
              ↓
┌─────────────────────────────────────┐
│ File: app/api/tasks/route.ts       │
│                                     │
│ export async function GET(         │
│   request: NextRequest              │
│   // No params needed!             │
│ ) {                                 │
│   const tasks = await getTasks()   │
│   return NextResponse.json(tasks)  │
│ }                                   │
└─────────────────────────────────────┘
```

---

### **Dynamic Route Flow**

```
Client Request: GET /api/tasks/123
                               ^^^
                               Dynamic segment
              ↓
┌─────────────────────────────────────────────┐
│ File: app/api/tasks/[id]/route.ts          │
│                     ^^^^                    │
│                     Folder name tells Next! │
│                                             │
│ export async function GET(                 │
│   request: NextRequest,                    │
│   { params }: { params: { id: string } }   │
│   //                      ^^               │
│   //                      Must match [id]  │
│ ) {                                         │
│   const taskId = params.id // "123"        │
│   const task = await getTask(taskId)       │
│   return NextResponse.json(task)           │
│ }                                           │
└─────────────────────────────────────────────┘
```

---

## 📊 Side-by-Side Comparison

| Aspect | Static Route | Dynamic Route |
|--------|-------------|---------------|
| **File Path** | `app/api/tasks/route.ts` | `app/api/tasks/[id]/route.ts` |
| **URL Example** | `/api/tasks` | `/api/tasks/123` |
| **Parameters** | `(request)` | `(request, { params })` |
| **Has params?** | ❌ No | ✅ Yes |
| **Use Case** | Collection operations | Single resource operations |
| **Examples** | Get all, Create new | Get one, Update one, Delete one |

---

## 🎓 When to Use Which

### **Use Static Routes For:**

✅ **Collection Operations**
- Get all items: `GET /api/tasks`
- Create new item: `POST /api/tasks`
- Batch operations: `DELETE /api/tasks` (delete all)

✅ **Search/Filter Operations**
- Search: `GET /api/tasks?search=keyword`
- Filter: `GET /api/tasks?status=completed`
- Paginate: `GET /api/tasks?page=2`

✅ **General Actions**
- Health check: `GET /api/health`
- Authentication: `POST /api/auth/login`
- Webhooks: `POST /api/webhooks`

---

### **Use Dynamic Routes For:**

✅ **Single Resource Operations**
- Get specific item: `GET /api/tasks/123`
- Update specific item: `PUT /api/tasks/123`
- Delete specific item: `DELETE /api/tasks/123`

✅ **Nested Resources**
- User's posts: `GET /api/users/5/posts`
- Post's comments: `GET /api/posts/42/comments/7`

✅ **Resource-Specific Actions**
- Archive task: `POST /api/tasks/123/archive`
- Share post: `POST /api/posts/42/share`

---

## 💡 RESTful API Conventions

Following REST conventions makes your API predictable:

```typescript
// Static route: app/api/tasks/route.ts
GET    /api/tasks              → Get all tasks
POST   /api/tasks              → Create new task

// Dynamic route: app/api/tasks/[id]/route.ts
GET    /api/tasks/123          → Get task 123
PUT    /api/tasks/123          → Update task 123
DELETE /api/tasks/123          → Delete task 123
PATCH  /api/tasks/123          → Partially update task 123
```

---

## 🛠️ Multiple Dynamic Segments

You can have multiple dynamic segments in a single route:

### **File Structure**

```
app/
  api/
    users/
      [userId]/
        posts/
          [postId]/
            route.ts
```

### **Function Signature**

```typescript
// File: app/api/users/[userId]/posts/[postId]/route.ts

export async function GET(
  request: NextRequest,
  { params }: {
    params: {
      userId: string;   // From [userId]
      postId: string;   // From [postId]
    }
  }
) {
  const { userId, postId } = params;

  const post = await getUserPost(userId, postId);
  return NextResponse.json(post);
}
```

### **URL Matching**

```
URL: /api/users/5/posts/42

Matches: app/api/users/[userId]/posts/[postId]/route.ts

params = {
  userId: "5",
  postId: "42"
}
```

---

## ⚠️ Common Mistakes

### **Mistake 1: Using params in Static Route**

```typescript
// ❌ WRONG - Static route doesn't have params
// File: app/api/tasks/route.ts
export async function POST(
  request: NextRequest,
  { params }: { params: { title: string } } // ❌ No params!
) { }

// ✅ CORRECT - Only request parameter
export async function POST(request: NextRequest) {
  const body = await request.json();
  const { title } = body;
}
```

---

### **Mistake 2: Not Using params in Dynamic Route**

```typescript
// ❌ WRONG - Dynamic route needs params
// File: app/api/tasks/[id]/route.ts
export async function GET(request: NextRequest) {
  // How do I get the ID? 🤔
}

// ✅ CORRECT - Use params
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const taskId = params.id; // ✅ Access the ID
}
```

---

### **Mistake 3: Wrong Param Name**

```typescript
// ❌ WRONG - param name must match folder name
// File: app/api/tasks/[id]/route.ts
export async function GET(
  request: NextRequest,
  { params }: { params: { taskId: string } } // ❌ Should be "id"
) { }

// ✅ CORRECT - Match the [id] folder name
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } } // ✅ Matches [id]
) { }
```

---

## 🎯 Decision Tree

```
Do you need to access a specific resource by ID?
    │
    ├─ YES → Use DYNAMIC route
    │         Example: /api/tasks/[id]
    │         Function: GET(request, { params })
    │
    └─ NO → Use STATIC route
            Example: /api/tasks
            Function: GET(request)
```

---

## 📋 Complete CRUD Example

### **Static Route: Collections**

```typescript
// File: app/api/tasks/route.ts

// GET all tasks
export async function GET(request: NextRequest) {
  const tasks = await getTasks();
  return NextResponse.json(tasks);
}

// POST create task
export async function POST(request: NextRequest) {
  const body = await request.json();
  const task = await createTask(body);
  return NextResponse.json(task, { status: 201 });
}
```

---

### **Dynamic Route: Single Resource**

```typescript
// File: app/api/tasks/[id]/route.ts

// GET single task
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const task = await getTask(params.id);
  return NextResponse.json(task);
}

// PUT update task
export async function PUT(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const body = await request.json();
  const updated = await updateTask(params.id, body);
  return NextResponse.json(updated);
}

// DELETE task
export async function DELETE(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  await deleteTask(params.id);
  return NextResponse.json({ success: true });
}
```

---

## 🎯 Key Takeaways

1. ✅ **Static routes** = Fixed paths, ONE parameter
2. ✅ **Dynamic routes** = Variable paths with `[brackets]`, TWO parameters
3. ✅ **Static**: `GET(request: NextRequest)`
4. ✅ **Dynamic**: `GET(request: NextRequest, { params })`
5. ✅ **Param names must match folder names**: `[id]` → `params.id`
6. ✅ **Follow REST conventions** for predictable APIs
7. ✅ **Use static for collections, dynamic for single resources**

---

## 📝 Practice Exercise

Set up the correct routes for a blog API:

**Requirements:**
- Get all posts
- Get single post
- Create new post
- Update post
- Delete post
- Get comments for a post

<details>
<summary>Click for solution</summary>

```typescript
// File: app/api/posts/route.ts (STATIC)

// GET all posts
export async function GET(request: NextRequest) {
  const posts = await getPosts();
  return NextResponse.json(posts);
}

// POST create post
export async function POST(request: NextRequest) {
  const body = await request.json();
  const post = await createPost(body);
  return NextResponse.json(post, { status: 201 });
}

// ─────────────────────────────────────────────

// File: app/api/posts/[id]/route.ts (DYNAMIC)

// GET single post
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const post = await getPost(params.id);
  return NextResponse.json(post);
}

// PUT update post
export async function PUT(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const body = await request.json();
  const updated = await updatePost(params.id, body);
  return NextResponse.json(updated);
}

// DELETE post
export async function DELETE(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  await deletePost(params.id);
  return NextResponse.json({ success: true });
}

// ─────────────────────────────────────────────

// File: app/api/posts/[id]/comments/route.ts (NESTED)

// GET comments for a post
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const comments = await getPostComments(params.id);
  return NextResponse.json(comments);
}
```

</details>

---

_Understanding static vs dynamic routes is fundamental to structuring your Next.js API correctly._
