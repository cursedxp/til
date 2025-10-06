# TypeScript Function Return Types

## What does `(category: string): BlogPost[]` mean?

This is a TypeScript **function type signature**. It defines:

1. **Parameters**: `(category: string)` - The function accepts one parameter named `category` of type `string`
2. **Return Type**: `: BlogPost[]` - The function returns an **array** of `BlogPost` objects

## Type Safety Guarantee

By specifying `BlogPost[]` as the return type, you are guaranteeing:

- ✅ The returned value **must be an array**
- ✅ Every element in the array **must be of type `BlogPost`**
- ✅ TypeScript will **check this at compile-time** (catches errors while coding)

## Example

```typescript
function getPostsByCategory(category: string): BlogPost[] {
  // ✅ Correct - Returns an array of BlogPost objects
  return [
    { title: "Post 1", content: "...", category: "tech" },
    { title: "Post 2", content: "...", category: "tech" }
  ];

  // ❌ Error - Cannot return a string
  // return "some string";

  // ❌ Error - Object doesn't match BlogPost structure
  // return [{ wrongField: "data" }];

  // ❌ Error - Cannot return a single object, must be an array
  // return { title: "Post 1", content: "..." };
}
```

## Usage

```typescript
const techPosts = getPostsByCategory("technology"); // Returns BlogPost[]
```

## Benefits

This makes your code **type-safe**:
- Prevents using wrong types
- Provides autocomplete/IntelliSense
- Catches errors before runtime
- Makes code more maintainable and self-documenting
