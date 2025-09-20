# Instruction Syntax and Workflows - The Brain of AI Frameworks

## What Are Instructions?

Instructions are the **step-by-step guides** that tell AI agents exactly how to execute complex tasks. They transform vague user requests into precise, repeatable workflows.

## From Simple to Sophisticated

### Level 1: Basic Text Instructions
```
Please review this code for security issues and bugs.
Focus on SQL injection and XSS vulnerabilities.
```

### Level 2: Structured Markdown
```markdown
## Code Review Process
1. Scan for security vulnerabilities
2. Check for performance issues
3. Verify coding standards
4. Generate report
```

### Level 3: XML Workflows (Agent OS style)
```xml
<process_flow>
  <step number="1" subagent="security-scanner">
    <action>Scan for vulnerabilities</action>
    <output>security-report.md</output>
  </step>

  <step number="2" subagent="performance-checker">
    <action>Analyze performance</action>
    <output>performance-report.md</output>
  </step>
</process_flow>
```

## Agent OS XML Syntax Explained

### Why XML?

XML tags create **clear boundaries** and **structure** that AI agents can reliably parse:

```xml
<step number="1" subagent="context-fetcher" name="gather_info">
  <action>Read specification files</action>
  <output>requirements.md</output>
</step>
```

This tells the AI:
- **Step order**: Do this first (number="1")
- **Who**: Use context-fetcher agent
- **What**: Read specification files
- **Result**: Save as requirements.md

### Real Agent OS Pattern
```xml
<process_flow>
  <step number="1" subagent="context-fetcher">
    <action>READ @spec.md</action>
    <output>requirements</output>
  </step>

  <step number="2" subagent="file-creator">
    <input>@requirements</input>
    <action>CREATE component files</action>
    <template>react-component.template</template>
  </step>

  <step number="3" subagent="test-writer">
    <action>GENERATE tests</action>
    <coverage>80%</coverage>
  </step>
</process_flow>
```

## Common Workflow Patterns

### 1. Sequential Steps
```xml
<sequential>
  <step>Validate input</step>
  <step>Process data</step>
  <step>Generate output</step>
  <step>Run tests</step>
</sequential>
```

### 2. Conditional Logic
```xml
<if condition="file_exists('package.json')">
  <action>Use npm commands</action>
</if>
<else>
  <action>Use manual setup</action>
</else>
```

### 3. Parallel Tasks
```xml
<parallel>
  <task>Build frontend</task>
  <task>Build backend</task>
  <task>Generate docs</task>
</parallel>
```

### 4. File Templates
```xml
<create file="component.tsx">
  <from_template>react-component.template</from_template>
  <variables>
    <name>{{component_name}}</name>
    <props>{{props_list}}</props>
  </variables>
</create>
```

## Building Your Own Instructions

### Basic Instruction Structure
```yaml
---
name: build-feature
description: Creates a new feature from spec
version: 1.0.0
---

# Build Feature Workflow

<process_flow>
  <step number="1">
    Read the feature specification
  </step>
  <step number="2">
    Generate implementation code
  </step>
  <step number="3">
    Create unit tests
  </step>
</process_flow>
```

### Pre and Post Checks
```xml
<pre_flight_check>
  <verify>Git working directory is clean</verify>
  <verify>Dependencies are installed</verify>
</pre_flight_check>

<post_flight_check>
  <verify>All tests pass</verify>
  <verify>Code passes linting</verify>
</post_flight_check>
```

## Alternative Instruction Formats

### JSON Format
```json
{
  "workflow": "create-api",
  "steps": [
    {
      "agent": "api-builder",
      "action": "create-endpoint",
      "input": "spec.md",
      "output": "api.js"
    },
    {
      "agent": "test-writer",
      "action": "create-tests",
      "input": "api.js",
      "output": "api.test.js"
    }
  ]
}
```

### YAML Format
```yaml
workflow: create-component
steps:
  - agent: analyzer
    action: read-requirements
    input: spec.md

  - agent: generator
    action: create-component
    framework: react
    typescript: true
```

### Simple Text Format
```
WORKFLOW: Build Feature

STEP 1: Read specification
  - Agent: reader
  - Input: spec.md
  - Output: requirements

STEP 2: Generate code
  - Agent: coder
  - Input: requirements
  - Output: feature.js
```

## Error Handling in Workflows

### Try-Catch Pattern
```xml
<try>
  <step>Run test suite</step>
</try>
<catch>
  <step>Generate error report</step>
  <step>Notify developer</step>
</catch>
```

### Retry Logic
```xml
<step retry="3" delay="5000">
  <action>Deploy to server</action>
  <on_failure>
    <action>Rollback deployment</action>
  </on_failure>
</step>
```

## Context and Variable Management

### Passing Data Between Steps
```xml
<step produces="user_input">
  <action>Collect requirements</action>
</step>

<step requires="user_input">
  <action>Generate code using {{user_input}}</action>
</step>
```

### Global Variables
```xml
<variables>
  <project_root>./src</project_root>
  <test_dir>./tests</test_dir>
</variables>

<step>
  <action>Create file in {{project_root}}/components</action>
</step>
```

## Best Practices

### 1. Clear Step Names
```xml
<!-- Good -->
<step name="validate_user_schema">

<!-- Bad -->
<step name="step1">
```

### 2. Explicit Inputs and Outputs
```xml
<step>
  <input>user-requirements.md</input>
  <action>Generate component</action>
  <output>Button.tsx</output>
</step>
```

### 3. Document Complex Logic
```xml
<step>
  <!-- This step validates that the database schema
       matches the API specification before deployment -->
  <action>Validate schema compatibility</action>
</step>
```

### 4. Handle Edge Cases
```xml
<step>
  <action>Deploy to production</action>
  <if condition="environment == 'staging'">
    <action>Run smoke tests first</action>
  </if>
  <on_error>
    <action>Automatically rollback</action>
  </on_error>
</step>
```

## Testing Your Instructions

### Validation Checklist
- ✅ Each step has clear action
- ✅ Dependencies are explicit
- ✅ Error cases are handled
- ✅ Outputs are specified
- ✅ Variables are defined

### Test with Simple Cases
Before complex workflows, test with:
1. Single-step instructions
2. Two-step sequential flow
3. Simple conditional logic
4. Basic error handling

## Common Instruction Patterns

### CRUD Operations
```xml
<template name="crud-entity">
  <step>Create model file</step>
  <step>Create controller file</step>
  <step>Create route definitions</step>
  <step>Generate unit tests</step>
</template>
```

### Feature Development
```xml
<workflow name="new-feature">
  <step>Analyze requirements</step>
  <step>Design architecture</step>
  <step>Implement feature</step>
  <step>Write tests</step>
  <step>Update documentation</step>
</workflow>
```

### Code Review
```xml
<workflow name="review-code">
  <step>Check security issues</step>
  <step>Validate performance</step>
  <step>Verify standards compliance</step>
  <step>Generate review report</step>
</workflow>
```

## Key Takeaways

1. **Instructions define workflows** - They break complex tasks into manageable steps

2. **XML provides structure** - Clear syntax that AI agents can reliably parse

3. **Start simple** - Build complexity gradually with basic patterns

4. **Handle errors** - Always plan for things that can go wrong

5. **Use templates** - Reusable patterns for common tasks

6. **Test thoroughly** - Validate instructions work as expected

7. **Document clearly** - Help future you understand the workflow

Instructions are the **recipe book** for your AI agents. Well-written instructions ensure consistent, reliable results every time.