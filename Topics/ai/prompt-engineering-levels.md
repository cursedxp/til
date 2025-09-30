# Prompt Engineering Levels

## Understanding When to Use Markup

Not all prompts need special syntax. Understanding the three levels of prompt engineering helps you choose the right approach for your needs.

## Level 1: Regular User Prompts

**When to use:** Everyday interactions with AI

**Approach:** Natural language, no special syntax required

**Examples:**
```
"Help me fix this bug"
"Explain how this function works"
"Refactor this code to be more efficient"
"What's wrong with my React component?"
```

**Best for:**
- Simple questions
- One-off conversations
- Debugging sessions
- General explanations
- Quick tasks

**Key point:** AI models understand natural language perfectly. No markup needed!

---

## Level 2: Structured User Prompts

**When to use:** Complex requests that benefit from organization

**Approach:** Organized plain text with clear sections

**Examples:**
```
Task: Implement user authentication
Requirements:
- Use JWT tokens
- Include refresh token mechanism
- Add rate limiting
- Write unit tests

Context:
- Next.js 14 application
- PostgreSQL database
- Existing user model in place

Expected Output:
- Working authentication flow
- Test coverage > 80%
- Documentation in README
```

**Best for:**
- Multi-step tasks
- Complex requirements
- Project work
- When context matters
- Collaborative work

**Key point:** Structure improves clarity but still uses plain language.

---

## Level 3: System-Level Instructions

**When to use:** Building reusable AI systems and templates

**Approach:** Professional prompt engineering with semantic markup

**Examples:**
```xml
<role>You are a security-focused code reviewer</role>

<context>Production financial application</context>

<constraints>
  - Zero tolerance for security vulnerabilities
  - Maintain backward compatibility
  - Follow OWASP guidelines
</constraints>

<expertise>
  - React/TypeScript
  - Node.js security
  - PostgreSQL
</expertise>

<task>{{USER_REQUEST}}</task>

<output_format>
  1. Security assessment
  2. Code quality review
  3. Specific recommendations
  4. Risk rating
</output_format>
```

**Best for:**
- Agent OS instructions
- Reusable AI templates
- Team standardization
- Complex multi-step workflows
- Consistent AI behavior across projects
- Audit and compliance requirements

**Key point:** Markup provides clear boundaries and semantic meaning for system-level configuration.

---

## Quick Decision Guide

### Choose Level 1 (Natural Language) When:
- ✅ Simple, one-off requests
- ✅ Debugging or troubleshooting
- ✅ Questions and explanations
- ✅ Quick edits or fixes

### Choose Level 2 (Structured Plain Text) When:
- ✅ Complex multi-part tasks
- ✅ Need to provide context
- ✅ Multiple requirements
- ✅ Collaborative work

### Choose Level 3 (Markup) When:
- ✅ Building reusable templates
- ✅ System instructions (like Agent OS)
- ✅ Team standardization needed
- ✅ Complex workflows
- ✅ Consistent behavior required

---

## Common Mistakes

### ❌ Over-Engineering Simple Requests
```xml
<!-- Don't do this for simple questions -->
<task>
  <type>question</type>
  <content>What is a React hook?</content>
  <format>detailed_explanation</format>
</task>

<!-- Just do this -->
"What is a React hook?"
```

### ❌ Under-Structuring Complex Tasks
```
<!-- Too vague for complex work -->
"Build an authentication system"

<!-- Better -->
Task: Implement user authentication
Requirements:
- JWT tokens
- Refresh token mechanism
- Rate limiting
Context:
- Next.js 14
- PostgreSQL with Prisma
Expected Output:
- Working auth flow
- Tests with >80% coverage
```

---

## Evolution of Your Usage

### Starting Out
- **Use Level 1** for everything
- Learn what AI can do naturally
- Build intuition for prompting

### Growing Experience
- **Add Level 2** for complex tasks
- Structure helps you think clearly
- Better results with context

### Professional Usage
- **Use Level 3** for templates
- Build reusable systems
- Standardize across team

---

## Key Takeaways

1. **Default to natural language** - AI understands plain English
2. **Add structure when needed** - Helps with complex tasks
3. **Use markup for systems** - Not everyday interactions
4. **Match level to task complexity** - Don't over-engineer
5. **Clarity is the goal** - Not syntactic complexity

## Related Topics
- [XML Markup Patterns in AI](./xml-markup-patterns.md)
- [Prompt Engineering Best Practices](./prompt-engineering-best-practices.md)
- [Agent OS Instruction Syntax](../agentos/agent-os_06-instruction-syntax-and-workflows.md)