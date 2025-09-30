# Prompt Engineering Best Practices

## Core Principles

The goal of prompt engineering is **communication clarity**, not syntactic complexity. Use the simplest approach that achieves your objective.

## 1. Be Specific

### ❌ Bad: Vague Requests
```xml
<task>Fix the bug</task>
```

### ✅ Good: Specific Details
```xml
<task>
  Fix null pointer exception in UserAuth.validate()
  that occurs when email field is empty
</task>
```

**Why it matters:**
- Specific requests get specific solutions
- Less back-and-forth clarification
- Faster problem resolution

---

## 2. Provide Context

### ❌ Bad: No Context
```
"Add authentication to the app"
```

### ✅ Good: Rich Context
```xml
<task>Add user authentication</task>

<context>
  - Framework: Next.js 14 with App Router
  - Database: PostgreSQL with Prisma ORM
  - Current state: User model exists, no auth implemented
  - Authentication: NextAuth.js preferred
  - Deployment: Vercel
</context>

<constraints>
  - Use existing User model
  - Support GitHub OAuth
  - Add JWT token refresh
</constraints>
```

**Why it matters:**
- AI makes better decisions with context
- Aligns solution with your tech stack
- Avoids incompatible suggestions

---

## 3. Define Success Criteria

### ❌ Bad: Open-Ended
```
"Make it faster"
```

### ✅ Good: Measurable Goals
```xml
<success_criteria>
  - All existing tests pass
  - New tests cover edge cases
  - API response time: < 200ms (currently 800ms)
  - No TypeScript errors
  - No new ESLint warnings
  - Bundle size increase: < 10kb
</success_criteria>
```

**Why it matters:**
- Clear definition of "done"
- Measurable outcomes
- Prevents scope creep

---

## 4. Include Examples

### ❌ Bad: Abstract Description
```
"Follow our coding standards"
```

### ✅ Good: Concrete Examples
```xml
<examples>
  <good>
    const result = await fetchUser(id);
    if (!result) {
      throw new UserNotFoundError(id);
    }
    return result;
  </good>

  <bad>
    const result = fetchUser(id); // Missing await
    return result;
  </bad>
</examples>

<coding_standards>
  - Always use async/await, never .then()
  - Throw custom errors, not generic Error
  - Validate inputs at function entry
</coding_standards>
```

**Why it matters:**
- Examples are clearer than descriptions
- Shows exactly what you want
- Prevents misinterpretation

---

## 5. Break Down Complex Tasks

### ❌ Bad: Monolithic Request
```
"Build a complete e-commerce system with user management,
product catalog, shopping cart, checkout, payments, and admin panel"
```

### ✅ Good: Step-by-Step Approach
```xml
<task>Implement shopping cart functionality</task>

<steps>
  1. Create cart data model
  2. Add cart context provider
  3. Implement add/remove/update actions
  4. Create cart UI components
  5. Add local storage persistence
  6. Write integration tests
</steps>

<note>
  This is phase 2 of 6 for the e-commerce system.
  Authentication and product catalog already complete.
</note>
```

**Why it matters:**
- Manageable chunks
- Clear progression
- Easier to review and test

---

## 6. Specify Output Format

### ❌ Bad: No Format Guidance
```
"Review this code"
```

### ✅ Good: Structured Output
```xml
<task>Review authentication implementation</task>

<output_format>
  1. Security Assessment
     - Rating: Critical/High/Medium/Low
     - Specific vulnerabilities found

  2. Code Quality Review
     - Type safety issues
     - Error handling gaps
     - Performance concerns

  3. Recommendations
     - Must fix (blocks deployment)
     - Should fix (before release)
     - Consider (future improvement)

  4. Testing Suggestions
     - Missing test cases
     - Edge cases to cover
</output_format>
```

**Why it matters:**
- Consistent, actionable feedback
- Easy to parse and act on
- Nothing important gets missed

---

## 7. Handle Edge Cases

### ❌ Bad: Happy Path Only
```xml
<task>Fetch user data and display profile</task>
```

### ✅ Good: Consider Edge Cases
```xml
<task>Fetch user data and display profile</task>

<edge_cases>
  - User not found (404)
  - Network timeout
  - Malformed API response
  - User has no profile data
  - User profile is incomplete
  - API returns rate limit error
</edge_cases>

<error_handling>
  - Show user-friendly error messages
  - Log errors for debugging
  - Provide retry mechanism
  - Fallback to cached data if available
</error_handling>
```

**Why it matters:**
- Production-ready code
- Better user experience
- Fewer surprises in production

---

## 8. Learn from Iterations

### Track What Works
```markdown
## Prompt Iteration Log

### Attempt 1: Too Vague
"Add error handling"
Result: Generic try-catch blocks

### Attempt 2: More Specific
"Add error handling for API calls with custom error types"
Result: Better, but missing retry logic

### Attempt 3: Complete
"Add error handling for API calls with:
- Custom error types for each failure mode
- Exponential backoff retry (3 attempts)
- User-friendly error messages
- Sentry error logging"
Result: ✅ Production-ready implementation
```

**Why it matters:**
- Learn what works for your use case
- Build library of effective prompts
- Continuous improvement

---

## Tools for Prompt Engineering

### Testing Frameworks
- **Promptfoo**: Test prompt variations and compare results
- **LangSmith**: Debug and optimize prompts with detailed tracing
- **Helicone**: Monitor prompt performance and costs

### Management Systems
- **PromptBase**: Marketplace for effective prompts
- **Dust**: Build and deploy prompt chains
- **LangChain**: Orchestrate complex prompt workflows

### Evaluation Tools
- **OpenAI Evals**: Test prompt quality systematically
- **Weights & Biases**: Track prompt experiments
- **Humanloop**: A/B test different prompt versions

---

## Common Pitfalls to Avoid

### 1. Over-Engineering Simple Tasks
```xml
<!-- Don't do this for simple questions -->
<system>
  <role>Expert explainer</role>
  <task>Explain React hooks</task>
  <output_format>Detailed with examples</output_format>
</system>

<!-- Just do this -->
"Explain React hooks with examples"
```

### 2. Assuming AI Knows Your Context
```
<!-- Too vague -->
"Fix the login issue"

<!-- Better -->
"Fix the login issue where users get 'Invalid credentials'
error even with correct password. This started after upgrading
NextAuth from v4 to v5."
```

### 3. Not Iterating on Prompts
First attempt rarely perfect. Refine based on results.

### 4. Ignoring Token Limits
Keep prompts concise. More words ≠ better results.

### 5. Not Testing Prompts
Test with various inputs to ensure consistency.

---

## The Evolution of Prompt Engineering

### Early Days (GPT-3 era)
```
You are a helpful assistant.
Rules:
1. Be polite
2. Be accurate
3. Be concise
```

### Current Generation (GPT-4/Claude 3)
```xml
<system>
  <role>Expert TypeScript developer</role>
  <expertise>
    - React ecosystem
    - Node.js backend
    - PostgreSQL
  </expertise>
  <constraints>
    <performance>Optimize for sub-100ms response</performance>
    <security>Follow OWASP Top 10</security>
  </constraints>
</system>
```

### Emerging Patterns
```yaml
# YAML-based configuration
agent:
  role: code_reviewer
  focus: [security, performance, maintainability]
  severity_levels:
    critical: blocks deployment
    warning: should fix
    info: consider improving
```

---

## The Future of Prompt Engineering

As AI models evolve, we're seeing:
- ✨ **Less need for explicit markup** - Models understand natural language better
- ✨ **More natural language understanding** - Complex requests work with plain text
- ✨ **Context-aware responses** - AI infers intent without extensive tags
- ✨ **Automatic instruction inference** - AI fills in gaps intelligently

However, **structured markup remains valuable for:**
- 🎯 Reproducible system behavior
- 🎯 Team standardization
- 🎯 Complex multi-agent systems
- 🎯 Audit and compliance requirements
- 🎯 Fine-grained control over outputs

---

## Key Takeaways

1. **Clarity over complexity** - Simple, clear prompts beat complex ones
2. **Context is critical** - Provide relevant background information
3. **Be specific** - Vague requests get vague responses
4. **Define success** - Know what "done" looks like
5. **Include examples** - Show, don't just tell
6. **Iterate and improve** - First attempt rarely perfect
7. **Match complexity to task** - Don't over-engineer simple requests
8. **Test systematically** - Verify prompts work consistently

---

## Template for Effective Prompts

```xml
<task>
  [Clear, specific description of what needs to be done]
</task>

<context>
  - Tech stack: [Your technologies]
  - Current state: [What exists now]
  - Constraints: [Limitations or requirements]
</context>

<success_criteria>
  - [Measurable outcome 1]
  - [Measurable outcome 2]
  - [Measurable outcome 3]
</success_criteria>

<examples>
  <good>
    [Example of what you want]
  </good>
  <avoid>
    [Example of what you don't want]
  </avoid>
</examples>

<output_format>
  [How you want the response structured]
</output_format>
```

---

## Related Topics
- [Prompt Engineering Levels](./prompt-engineering-levels.md)
- [XML Markup Patterns](./xml-markup-patterns.md)
- [Agent OS Instruction Syntax](../agentos/agent-os_06-instruction-syntax-and-workflows.md)
- [Understanding AI Frameworks](../agentos/agent-os_04-understanding-ai-frameworks.md)