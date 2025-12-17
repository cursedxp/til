# TypeScript Utility Types - Omit, Pick, Partial, and More

**Date**: 2025-10-17
**Topic**: Understanding TypeScript's built-in utility types for transforming existing types

---

## 🎯 What Are Utility Types?

**Utility Types** are built-in TypeScript helpers that let you **transform existing types** into new types without duplicating code.

Think of them as "type transformers" - you give them an existing type, and they create a modified version.

---

## 🔧 The Most Common Utility Types

### 1. **`Omit<Type, Keys>`** - Remove Properties

**Purpose**: Create a new type by **removing** specific properties from an existing type.

#### **Syntax**:
```typescript
Omit<OriginalType, "propertyToRemove">
```

#### **Example**:
```typescript
interface Task {
  id: string;
  title: string;
  description?: string;
  completed: boolean;
  priority?: "low" | "medium" | "high";
}

// Create a type without 'id' (for creating new tasks)
type CreateTaskInput = Omit<Task, "id">;

// Equivalent to:
type CreateTaskInput = {
  title: string;
  description?: string;
  completed: boolean;
  priority?: "low" | "medium" | "high";
}
```

#### **Real-World Use Case**:
```typescript
// When creating a new task, user shouldn't provide ID
export const createTask = async (taskData: Omit<Task, "id">): Promise<Task> => {
  const newId = generateId(); // Server generates ID
  return { id: newId, ...taskData };
};
```

#### **Remove Multiple Properties**:
```typescript
// Remove both 'id' and 'createdAt'
type UpdateTaskInput = Omit<Task, "id" | "createdAt">;
```

---

### 2. **`Pick<Type, Keys>`** - Keep Only Specific Properties

**Purpose**: Create a new type by **keeping only** specific properties from an existing type.

#### **Syntax**:
```typescript
Pick<OriginalType, "propertyToKeep">
```

#### **Example**:
```typescript
interface User {
  id: string;
  username: string;
  email: string;
  password: string;
  firstName: string;
  lastName: string;
  createdAt: Date;
}

// Only keep safe properties for public display
type PublicUser = Pick<User, "id" | "username" | "firstName" | "lastName">;

// Equivalent to:
type PublicUser = {
  id: string;
  username: string;
  firstName: string;
  lastName: string;
}
```

#### **Real-World Use Case**:
```typescript
// Return only safe user data to frontend
export async function getPublicUser(userId: string): Promise<PublicUser> {
  const user = await database.users.findById(userId);

  // TypeScript ensures you only return allowed properties
  return {
    id: user.id,
    username: user.username,
    firstName: user.firstName,
    lastName: user.lastName
  };
}
```

---

### 3. **`Partial<Type>`** - Make All Properties Optional

**Purpose**: Create a new type where **all properties are optional**.

#### **Syntax**:
```typescript
Partial<OriginalType>
```

#### **Example**:
```typescript
interface Task {
  id: string;
  title: string;
  completed: boolean;
  priority: "low" | "medium" | "high";
}

// Make all properties optional
type PartialTask = Partial<Task>;

// Equivalent to:
type PartialTask = {
  id?: string;
  title?: string;
  completed?: boolean;
  priority?: "low" | "medium" | "high";
}
```

#### **Real-World Use Case**:
```typescript
// Allow updating any subset of task properties
export const updateTask = async (
  id: string,
  updates: Partial<Omit<Task, "id">>  // Can update any field except id
): Promise<Task> => {
  const task = await getTask(id);
  return { ...task, ...updates };  // Merge updates
};

// Usage:
updateTask("123", { title: "New Title" });  // ✅ Only update title
updateTask("123", { completed: true });     // ✅ Only update completed
updateTask("123", { title: "New", priority: "high" }); // ✅ Multiple fields
```

---

### 4. **`Required<Type>`** - Make All Properties Required

**Purpose**: Create a new type where **all properties are required** (opposite of `Partial`).

#### **Syntax**:
```typescript
Required<OriginalType>
```

#### **Example**:
```typescript
interface Task {
  id: string;
  title: string;
  description?: string;     // Optional
  priority?: "low" | "medium" | "high";  // Optional
}

// Make all properties required
type StrictTask = Required<Task>;

// Equivalent to:
type StrictTask = {
  id: string;
  title: string;
  description: string;      // Now required!
  priority: "low" | "medium" | "high";  // Now required!
}
```

#### **Real-World Use Case**:
```typescript
// Validation function that ensures all fields are present
function validateCompleteTask(task: Required<Task>): boolean {
  // TypeScript guarantees all properties exist
  return task.description.length > 0 && task.priority !== undefined;
}
```

---

### 5. **`Readonly<Type>`** - Make All Properties Read-Only

**Purpose**: Create a new type where **all properties are read-only** (can't be modified).

#### **Syntax**:
```typescript
Readonly<OriginalType>
```

#### **Example**:
```typescript
interface Task {
  id: string;
  title: string;
  completed: boolean;
}

// Make all properties read-only
type ImmutableTask = Readonly<Task>;

// Equivalent to:
type ImmutableTask = {
  readonly id: string;
  readonly title: string;
  readonly completed: boolean;
}
```

#### **Real-World Use Case**:
```typescript
// Return read-only tasks to prevent accidental mutations
export function getTasks(): Readonly<Task>[] {
  return tasks;
}

const tasks = getTasks();
tasks[0].title = "New Title";  // ❌ TypeScript error: Cannot assign to 'title' because it is a read-only property
```

---

### 6. **`Record<Keys, Type>`** - Create Object Type with Specific Keys

**Purpose**: Create an object type where all keys have the same value type.

#### **Syntax**:
```typescript
Record<KeyType, ValueType>
```

#### **Example**:
```typescript
// Create a type with string keys and number values
type TaskCounts = Record<string, number>;

// Equivalent to:
type TaskCounts = {
  [key: string]: number;
}

// Usage:
const counts: TaskCounts = {
  pending: 5,
  completed: 10,
  inProgress: 3
};
```

#### **With Specific Keys**:
```typescript
type Priority = "low" | "medium" | "high";

// Each priority must have a color
type PriorityColors = Record<Priority, string>;

// Equivalent to:
type PriorityColors = {
  low: string;
  medium: string;
  high: string;
}

// Usage:
const colors: PriorityColors = {
  low: "green",
  medium: "yellow",
  high: "red"
};
```

---

## 🎓 Combining Utility Types

You can **chain** utility types together for powerful transformations!

### **Example 1: Partial Omit**
```typescript
// Create type for updating tasks (no ID, all fields optional)
type UpdateTaskInput = Partial<Omit<Task, "id">>;

// Equivalent to:
type UpdateTaskInput = {
  title?: string;
  description?: string;
  completed?: boolean;
  priority?: "low" | "medium" | "high";
}
```

### **Example 2: Required Pick**
```typescript
// Only keep title and completed, but make them required
type MinimalTask = Required<Pick<Task, "title" | "completed">>;

// Equivalent to:
type MinimalTask = {
  title: string;
  completed: boolean;
}
```

### **Example 3: Readonly Partial**
```typescript
// Read-only task with optional properties
type ReadonlyPartialTask = Readonly<Partial<Task>>;

// Equivalent to:
type ReadonlyPartialTask = {
  readonly id?: string;
  readonly title?: string;
  readonly completed?: boolean;
  readonly priority?: "low" | "medium" | "high";
}
```

---

## 📊 Visual Comparison Table

| Utility Type | What It Does | Use Case |
|-------------|--------------|----------|
| `Omit<Task, "id">` | **Remove** specific properties | Creating new items (no ID yet) |
| `Pick<Task, "id" \| "title">` | **Keep only** specific properties | Public APIs (hide sensitive data) |
| `Partial<Task>` | Make **all properties optional** | Update operations (partial updates) |
| `Required<Task>` | Make **all properties required** | Validation (ensure completeness) |
| `Readonly<Task>` | Make **all properties read-only** | Prevent mutations |
| `Record<string, number>` | Create object with **uniform value type** | Dictionaries, maps, counts |

---

## 🎯 Practical Real-World Examples

### **Example 1: CRUD Operations**

```typescript
interface Task {
  id: string;
  title: string;
  description?: string;
  completed: boolean;
  priority: "low" | "medium" | "high";
  createdAt: Date;
  updatedAt: Date;
}

// CREATE - No ID or timestamps (server generates)
type CreateTaskInput = Omit<Task, "id" | "createdAt" | "updatedAt">;

// READ - Return full task
type TaskResponse = Task;

// UPDATE - ID required, other fields optional
type UpdateTaskInput = Partial<Omit<Task, "id" | "createdAt">> & { id: string };

// DELETE - Only need ID
type DeleteTaskInput = Pick<Task, "id">;
```

### **Example 2: API Response Types**

```typescript
interface User {
  id: string;
  email: string;
  password: string;
  firstName: string;
  lastName: string;
  role: "admin" | "user";
}

// Public profile (no sensitive data)
type PublicProfile = Pick<User, "id" | "firstName" | "lastName">;

// Login response (no password)
type LoginResponse = Omit<User, "password">;

// Registration input (no ID)
type RegisterInput = Omit<User, "id">;
```

### **Example 3: Form State Management**

```typescript
interface Product {
  id: string;
  name: string;
  price: number;
  description: string;
  stock: number;
}

// Form might have incomplete data as user types
type ProductFormState = Partial<Omit<Product, "id">>;

// Form validation requires all fields
type ValidatedProductForm = Required<Omit<Product, "id">>;
```

---

## 💡 When to Use Which?

### **Use `Omit` when:**
- Creating new records (remove `id`, `createdAt`, etc.)
- Hiding sensitive fields (remove `password`, `token`, etc.)
- Building input types for create operations

### **Use `Pick` when:**
- Returning public data (keep only safe fields)
- Building minimal types for specific operations
- Creating lightweight versions of large types

### **Use `Partial` when:**
- Update operations (allow partial updates)
- Form state (fields filled incrementally)
- Optional configuration objects

### **Use `Required` when:**
- Validation (ensure all fields present)
- Complete data processing
- Converting optional types to required

### **Use `Readonly` when:**
- Preventing mutations
- Immutable data structures
- Protecting configuration objects

### **Use `Record` when:**
- Creating dictionaries/maps
- Uniform key-value structures
- Indexed collections

---

## ⚠️ Common Mistakes

### **Mistake 1: Forgetting Union Types with Omit**
```typescript
// ❌ Wrong - string not union
type CreateInput = Omit<Task, "id, createdAt">;

// ✅ Correct - use union type
type CreateInput = Omit<Task, "id" | "createdAt">;
```

### **Mistake 2: Using Pick When Omit Is Clearer**
```typescript
interface LargeType {
  field1: string;
  field2: string;
  // ... 20 more fields
  sensitiveField: string;
}

// ❌ Harder to read - list all 20+ fields
type PublicType = Pick<LargeType, "field1" | "field2" /* ... */>;

// ✅ Clearer - just remove sensitive field
type PublicType = Omit<LargeType, "sensitiveField">;
```

### **Mistake 3: Not Combining with & for Complex Types**
```typescript
// ❌ Partial makes id optional too
type UpdateInput = Partial<Task>;

// ✅ ID required, rest optional
type UpdateInput = { id: string } & Partial<Omit<Task, "id">>;
```

---

## 🎯 Key Takeaways

1. ✅ **Utility types transform existing types** - no need to duplicate
2. ✅ **`Omit` removes**, **`Pick` keeps**, **`Partial` makes optional**
3. ✅ **Combine utility types** for powerful transformations
4. ✅ **DRY principle** - if Task changes, derived types update automatically
5. ✅ **Type safety** - TypeScript catches errors at compile time
6. ✅ **Best practice** - define input/output types using utilities

---

## 📝 Practice Questions

### **Question 1**: What's the result?
```typescript
interface User {
  id: string;
  name: string;
  email: string;
}

type Result = Omit<User, "email">;
```

### **Question 2**: Which is better for an update function?
```typescript
// A:
function updateTask(id: string, data: Partial<Task>): Promise<Task>

// B:
function updateTask(id: string, data: Partial<Omit<Task, "id">>): Promise<Task>
```

### **Question 3**: Create a type for this use case:
```
Need a type for creating a new blog post.
Blog post has: id, title, content, authorId, createdAt, updatedAt
User should only provide: title, content, authorId
```

---

_Utility types are your friends - they keep your code DRY and type-safe!_
