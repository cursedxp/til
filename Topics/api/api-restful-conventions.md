# RESTful API Conventions

**Date**: 2025-10-17
**Topic**: Understanding RESTful URL structure and conventions

---

## 🎯 The Problem

**Initial (Wrong) Approach**:
```
❌ /api/tasks/allTask       → Get all tasks
❌ /api/tasks/singleTask?id=123 → Get one task
```

**Why is this wrong?**
- Not following REST standards
- URLs contain action names (allTask, singleTask)
- Inconsistent with industry conventions
- Harder to maintain and understand

---

## ✅ The RESTful Solution

### **Correct URL Structure**:
```
GET    /api/tasks           → Get all tasks
GET    /api/tasks/123       → Get single task by ID
POST   /api/tasks           → Create new task
PUT    /api/tasks/123       → Update task 123
DELETE /api/tasks/123       → Delete task 123
```

---

## 🎓 REST Principles

### **1. Resource-Based URLs**

URLs represent **resources** (things), not **actions** (verbs):

```
❌ /api/getAllTasks
❌ /api/createTask
❌ /api/updateTask/123
❌ /api/deleteTask/123

✅ /api/tasks              (resource: tasks)
✅ /api/tasks/123          (resource: specific task)
```

### **2. HTTP Methods Define Actions**

The **HTTP method** tells you what action to perform:

| Method | Action | Example |
|--------|--------|---------|
| GET | Read/Retrieve | `GET /api/tasks` → Get all tasks |
| POST | Create | `POST /api/tasks` → Create new task |
| PUT/PATCH | Update | `PUT /api/tasks/123` → Update task 123 |
| DELETE | Delete | `DELETE /api/tasks/123` → Delete task 123 |

### **3. URL Hierarchy Shows Relationships**

```
/api/tasks                    → All tasks
/api/tasks/123                → Specific task
/api/tasks/123/comments       → Comments for task 123
/api/tasks/123/comments/5     → Specific comment on task 123
```

---

## 📊 Query Parameters vs Route Parameters

### **Route Parameters** (Identifying Resources):
```
/api/tasks/123
           ^^^
Use for: Identifying a specific resource
```

### **Query Parameters** (Filtering/Options):
```
/api/tasks?status=completed&limit=10
           ^^^^^^^^^^^^^^^^^^^^^^^^^^
Use for: Filtering, sorting, pagination, optional parameters
```

### **Examples**:

```typescript
// ✅ Route params for resource identification
GET /api/users/123              → Get user 123
GET /api/posts/456              → Get post 456

// ✅ Query params for filtering/options
GET /api/tasks?status=completed → Get completed tasks
GET /api/users?page=2&limit=20  → Paginated users
GET /api/posts?author=john      → Posts by author
```

---

## 🛠️ Next.js Implementation

### **File Structure**:
```
app/api/tasks/
├── route.ts              ← /api/tasks (collection)
└── [id]/
    └── route.ts          ← /api/tasks/[id] (single resource)
```

### **Collection Route** (`/api/tasks/route.ts`):
```typescript
// GET all tasks
export async function GET() {
  const tasks = await getTasks();
  return NextResponse.json(tasks);
}

// POST create task
export async function POST(request: Request) {
  const body = await request.json();
  const newTask = await createTask(body);
  return NextResponse.json(newTask, { status: 201 });
}
```

### **Single Resource Route** (`/api/tasks/[id]/route.ts`):
```typescript
// GET single task
export async function GET(
  request: Request,
  { params }: { params: { id: string } }
) {
  const task = await getTask(params.id);
  return NextResponse.json(task);
}

// PUT update task
export async function PUT(
  request: Request,
  { params }: { params: { id: string } }
) {
  const body = await request.json();
  const updated = await updateTask(params.id, body);
  return NextResponse.json(updated);
}

// DELETE task
export async function DELETE(
  request: Request,
  { params }: { params: { id: string } }
) {
  await deleteTask(params.id);
  return NextResponse.json({ success: true });
}
```

---

## 🎯 Benefits of RESTful Design

1. **Predictable**: Developers know what to expect
2. **Self-documenting**: URLs describe the resource structure
3. **Scalable**: Easy to add new resources
4. **Standard**: Works with existing tools and libraries
5. **Cacheable**: GET requests can be cached by browsers/CDNs

---

## 📋 REST Checklist

- [ ] URLs represent resources (nouns), not actions (verbs)
- [ ] HTTP methods define the action (GET, POST, PUT, DELETE)
- [ ] Route params identify specific resources
- [ ] Query params for filtering/sorting/pagination
- [ ] Consistent naming (plural for collections: `/tasks` not `/task`)
- [ ] Proper status codes (200, 201, 404, 500, etc.)

---

## 💡 Common Patterns

### **Collections and Items**:
```
GET    /api/tasks           → List all tasks
POST   /api/tasks           → Create task
GET    /api/tasks/123       → Get task 123
PUT    /api/tasks/123       → Update task 123
DELETE /api/tasks/123       → Delete task 123
```

### **Nested Resources**:
```
GET    /api/users/5/tasks           → All tasks for user 5
GET    /api/users/5/tasks/123       → Task 123 for user 5
POST   /api/users/5/tasks           → Create task for user 5
```

### **Filtering & Pagination**:
```
GET /api/tasks?status=completed
GET /api/tasks?page=2&limit=20
GET /api/tasks?sort=created_at&order=desc
```

---

_RESTful design makes your API intuitive, maintainable, and professional._
