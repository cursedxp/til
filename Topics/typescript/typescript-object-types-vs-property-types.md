# TypeScript: Object Types vs Property Types

**Date**: 2025-10-17
**Topic**: Understanding the difference between typing a whole object vs typing individual properties

---

## 🎯 The Core Concept

In TypeScript, there's a crucial difference between:
- **Object Types**: Types that describe an entire object (e.g., `Task`, `User`)
- **Property Types**: Types for individual fields (e.g., `string`, `boolean`)

Confusing these leads to type errors and incorrect code!

---

## 📚 What Are Object Types?

**Object types** are interfaces or types that describe the **structure of an entire object**.

### **Example: Task Object Type**

```typescript
// This is an OBJECT TYPE
interface Task {
  id: string;
  title: string;
  description: string;
  completed: boolean;
  priority: 'low' | 'medium' | 'high';
}
```

**What it describes**: The **entire task object** with all its properties

**How to use it**:
```typescript
// ✅ Correct - annotating a whole object
const myTask: Task = {
  id: "1",
  title: "Buy milk",
  description: "Get 2 gallons",
  completed: false,
  priority: "medium"
};
```

---

## 📚 What Are Property Types?

**Property types** are the types of **individual fields** within an object.

### **Example: Property Types in Task**

```typescript
interface Task {
  id: string;              // ← Property type: string
  title: string;           // ← Property type: string
  description: string;     // ← Property type: string
  completed: boolean;      // ← Property type: boolean
  priority: 'low' | 'medium' | 'high'; // ← Property type: union
}
```

**What they describe**: Each individual field's type

**How to use them**:
```typescript
// ✅ Correct - annotating individual properties
const taskId: string = "1";
const taskTitle: string = "Buy milk";
const isCompleted: boolean = false;
const taskPriority: 'low' | 'medium' | 'high' = "medium";
```

---

## ❌ The Common Mistake

### **Wrong: Using Object Type as Property Type**

```typescript
// ❌ WRONG - Using Task (object type) for individual properties
interface WrongParams {
  title: Task;        // ❌ Task is the WHOLE object, not a string!
  description: Task;  // ❌ description is a string, not a Task object!
  completed: Task;    // ❌ completed is a boolean, not a Task object!
}
```

**Why it's wrong:**
- `Task` represents the **entire task object**
- Each property needs its **own specific type**
- You're trying to assign an object type where a primitive type belongs

---

### **Right: Using Property Types Correctly**

```typescript
// ✅ CORRECT - Using proper property types
interface CorrectParams {
  title: string;      // ✅ title is a string
  description: string; // ✅ description is a string
  completed: boolean; // ✅ completed is a boolean
}
```

---

## 🔍 Visual Comparison

### **Object Type Usage**

```typescript
interface Task {
  id: string;
  title: string;
  completed: boolean;
}

// ✅ Use object type for WHOLE object
const task: Task = {
  id: "1",
  title: "Buy milk",
  completed: false
};

// ✅ Function parameter
function saveTask(task: Task) {
  // task is the whole object
}

// ✅ Function return type
function getTask(): Task {
  return {
    id: "1",
    title: "Buy milk",
    completed: false
  };
}
```

---

### **Property Type Usage**

```typescript
interface Task {
  id: string;        // ← Property type
  title: string;     // ← Property type
  completed: boolean; // ← Property type
}

// ✅ Use property types for INDIVIDUAL fields
const id: string = "1";
const title: string = "Buy milk";
const completed: boolean = false;

// ✅ Function parameters (individual properties)
function updateTaskTitle(id: string, newTitle: string) {
  // id and newTitle are individual values
}
```

---

## 💡 When to Use Which

### **Use Object Types When:**

1. **Defining the structure of an object**
   ```typescript
   interface User {
     id: string;
     name: string;
     email: string;
   }
   ```

2. **Typing a whole object variable**
   ```typescript
   const user: User = {
     id: "1",
     name: "John",
     email: "john@example.com"
   };
   ```

3. **Function parameters accepting objects**
   ```typescript
   function saveUser(user: User) {
     // user is a whole object
   }
   ```

4. **API responses**
   ```typescript
   async function getUser(): Promise<User> {
     const response = await fetch('/api/users/1');
     return response.json(); // Returns User object
   }
   ```

---

### **Use Property Types When:**

1. **Defining individual fields in interfaces**
   ```typescript
   interface User {
     name: string;    // ← Property type
     age: number;     // ← Property type
     active: boolean; // ← Property type
   }
   ```

2. **Typing individual variables**
   ```typescript
   const userName: string = "John";
   const userAge: number = 25;
   const isActive: boolean = true;
   ```

3. **Function parameters for individual values**
   ```typescript
   function greet(name: string, age: number) {
     // name and age are individual values
   }
   ```

---

## 🎓 Real-World Examples

### **Example 1: Correct Object Type Usage**

```typescript
// types/task.ts
export interface Task {
  id: string;
  title: string;
  description: string;
  completed: boolean;
}

// lib/tasks.ts
export async function createTask(task: Task): Promise<Task> {
  //                              ^^^^^^^^^^          ^^^^
  //                              Object type         Object type
  const created = await db.tasks.create({ data: task });
  return created;
}

// Usage
const newTask: Task = {
  id: "1",
  title: "Buy milk",
  description: "Get 2 gallons",
  completed: false
};

await createTask(newTask);
```

---

### **Example 2: Correct Property Type Usage**

```typescript
// types/task.ts
export interface Task {
  id: string;              // ← Property type: string
  title: string;           // ← Property type: string
  description: string;     // ← Property type: string
  completed: boolean;      // ← Property type: boolean
}

// lib/tasks.ts
export async function updateTaskTitle(
  id: string,          // ← Property type: string
  newTitle: string     // ← Property type: string
): Promise<void> {
  await db.tasks.update({
    where: { id },
    data: { title: newTitle }
  });
}

// Usage
await updateTaskTitle("1", "Buy organic milk");
```

---

### **Example 3: API Route - Object Type**

```typescript
// app/api/tasks/route.ts
import { Task } from "@/app/types/task";

export async function POST(request: NextRequest) {
  const body = await request.json();

  // ✅ Type the whole object as Task
  const newTask: Task = {
    id: generateId(),
    title: body.title,
    description: body.description,
    completed: body.completed ?? false
  };

  const created = await createTask(newTask);
  return NextResponse.json(created);
}
```

---

### **Example 4: API Route - Property Types**

```typescript
// app/api/tasks/[id]/route.ts
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
  //                      ^^^^^^^^^^^^ ← Property type: string
) {
  const taskId: string = params.id; // ← Property type: string
  const task = await getTask(taskId);
  return NextResponse.json(task);
}
```

---

## 📊 Common Patterns

### **Pattern 1: Creating Related Object Types**

```typescript
// Base object type
interface Task {
  id: string;
  title: string;
  description: string;
  completed: boolean;
  createdAt: Date;
  updatedAt: Date;
}

// ✅ For creating (without id and timestamps)
type NewTask = Omit<Task, 'id' | 'createdAt' | 'updatedAt'>;

// ✅ For updating (all fields optional)
type TaskUpdate = Partial<NewTask>;

// ✅ For preview (only some fields)
type TaskPreview = Pick<Task, 'id' | 'title' | 'completed'>;
```

---

### **Pattern 2: Extracting Property Types**

```typescript
interface Task {
  id: string;
  priority: 'low' | 'medium' | 'high';
}

// ✅ Extract property types when needed
type TaskId = Task['id'];           // string
type TaskPriority = Task['priority']; // 'low' | 'medium' | 'high'

// Usage
function filterByPriority(priority: TaskPriority) {
  // priority must be 'low' | 'medium' | 'high'
}
```

---

### **Pattern 3: Combining Object and Property Types**

```typescript
interface Task {
  id: string;
  title: string;
  completed: boolean;
}

// ✅ Function that takes object AND individual properties
function updateTask(
  taskId: string,        // ← Property type
  updates: Partial<Task> // ← Object type (partial)
): Promise<Task> {       // ← Object type (return)
  return db.tasks.update({
    where: { id: taskId },
    data: updates
  });
}

// Usage
await updateTask("1", { title: "New title", completed: true });
//               ^^^  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
//               Property  Object
```

---

## ⚠️ Common Anti-Patterns

### **❌ Anti-Pattern 1: Object Type for Property**

```typescript
// ❌ Bad
interface Params {
  title: Task;        // Task is an object, not a string!
  description: Task;  // Wrong type level!
}

// ✅ Good
interface Params {
  title: string;      // Correct property type
  description: string; // Correct property type
}
```

---

### **❌ Anti-Pattern 2: Not Using Object Types**

```typescript
// ❌ Bad - no type safety
function createTask(title, description, completed) {
  return db.tasks.create({
    data: { title, description, completed }
  });
}

// ✅ Good - use object type
function createTask(task: NewTask): Promise<Task> {
  return db.tasks.create({ data: task });
}
```

---

### **❌ Anti-Pattern 3: Mixing Levels**

```typescript
// ❌ Bad - mixing object and property types incorrectly
interface MixedUp {
  task: Task;           // Object type
  taskTitle: string;    // Property from Task
  taskCompleted: boolean; // Property from Task
}

// ✅ Good - consistent level
interface TaskWithMetadata {
  task: Task;           // Just the object
  metadata: {
    viewCount: number;
    lastViewed: Date;
  }
}
```

---

## 🎯 Type Hierarchy

Understanding the hierarchy helps:

```
Object Type (Task)
    │
    ├── Property Type (id: string)
    ├── Property Type (title: string)
    ├── Property Type (description: string)
    ├── Property Type (completed: boolean)
    └── Property Type (priority: 'low' | 'medium' | 'high')
```

**Remember:**
- **Object type** = The container
- **Property types** = What's inside the container

---

## 📋 Quick Reference

| Scenario | Use This |
|----------|----------|
| Defining an object structure | Object Type (`interface Task`) |
| Typing a whole object variable | Object Type (`const task: Task`) |
| Typing an individual field | Property Type (`const title: string`) |
| Function accepting an object | Object Type (`function save(task: Task)`) |
| Function accepting individual values | Property Types (`function save(id: string, title: string)`) |
| API response | Object Type (`Promise<Task>`) |
| Object fields definition | Property Types (`title: string`) |

---

## 🎯 Key Takeaways

1. ✅ **Object types** describe entire objects (e.g., `Task`)
2. ✅ **Property types** describe individual fields (e.g., `string`, `boolean`)
3. ✅ **Never use object types as property types**
4. ✅ **Use object types for whole objects, property types for fields**
5. ✅ **Understand the hierarchy**: Object contains properties
6. ✅ **TypeScript utilities** help create related types (`Omit`, `Pick`, `Partial`)

---

## 📝 Practice Exercise

Identify and fix the type errors:

```typescript
// ❌ What's wrong here?
interface Task {
  id: string;
  title: string;
  completed: boolean;
}

interface TaskRequest {
  title: Task;        // ❓
  description: Task;  // ❓
  completed: Task;    // ❓
}

function createTask(request: TaskRequest): Task {
  return {
    id: generateId(),
    title: request.title,
    description: request.description,
    completed: request.completed
  };
}
```

<details>
<summary>Click for solution</summary>

**Problems:**
- Using `Task` (object type) as property types
- Properties should use primitive types

**Fixed version:**
```typescript
// ✅ Correct
interface Task {
  id: string;
  title: string;
  description: string;
  completed: boolean;
}

interface TaskRequest {
  title: string;      // ✅ Property type
  description: string; // ✅ Property type
  completed: boolean; // ✅ Property type
}

function createTask(request: TaskRequest): Task {
  return {
    id: generateId(),
    title: request.title,
    description: request.description,
    completed: request.completed
  };
}

// Or even better, use utility types:
type TaskRequest = Omit<Task, 'id'>;
```

</details>

---

_Understanding object types vs property types is essential for writing type-safe TypeScript code._
