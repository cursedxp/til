# YAML Basics and Syntax

## What is YAML?

YAML (YAML Ain't Markup Language) is a human-readable data serialization format designed for configuration files and data exchange. It prioritizes readability and uses indentation to represent structure.

## Key YAML Syntax Rules

### Basic Structure
- **Key-value pairs**: `key: value`
- **Indentation**: Use spaces (never tabs) to show hierarchy
- **Comments**: Start with `#`
- **No brackets needed**: Unlike JSON, YAML doesn't require brackets or quotes for most cases

### Data Types
```yaml
string: Hello World          # Strings (quotes optional)
number: 42                   # Numbers
boolean: true                # Booleans (true/false)
null_value: null            # Null values
```

### Lists
```yaml
# Method 1: Dash notation
fruits:
  - apple
  - banana
  - orange

# Method 2: Inline
fruits: [apple, banana, orange]
```

### Nested Structures
```yaml
database:
  host: localhost
  port: 5432
  credentials:
    username: admin
    password: secret123
```

## Where YAML is Commonly Used

1. **Docker & Kubernetes**: Container orchestration and configuration
2. **CI/CD Pipelines**: GitHub Actions, GitLab CI, Jenkins
3. **Application Configuration**: Spring Boot, Rails, Django settings
4. **Infrastructure as Code**: Ansible playbooks, CloudFormation templates
5. **API Specifications**: OpenAPI/Swagger definitions

## Real-World Examples

### Docker Compose
```yaml
version: '3.8'
services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
```

### GitHub Actions
```yaml
name: CI Pipeline
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - run: npm install
      - run: npm test
```

## YAML vs Other Formats

### YAML vs JSON
```yaml
# YAML - Clean and readable
person:
  name: John
  age: 30
  hobbies:
    - coding
    - gaming
```

```json
// JSON - More verbose
{
  "person": {
    "name": "John",
    "age": 30,
    "hobbies": ["coding", "gaming"]
  }
}
```

### YAML vs XML
YAML is significantly more concise than XML and doesn't require closing tags, making it more human-friendly for configuration purposes.

## Best Practices

1. **Consistent indentation**: Use 2 or 4 spaces consistently
2. **Quote special strings**: Use quotes for strings with special characters
3. **Avoid tabs**: Always use spaces for indentation
4. **Use comments**: Document complex configurations
5. **Validate your YAML**: Use online validators or linters

## Custom YAML Files

When creating your own YAML files (not framework-specific), you have complete freedom:

```yaml
# my-custom-config.yml
app_settings:
  theme: dark
  language: english

feature_flags:
  new_ui: true
  beta_features: false

custom_data:
  whatever:
    you:
      want: "Total freedom in structure!"
```

## Key Takeaway

YAML is everywhere in modern development because it strikes the perfect balance between human readability and machine parseability. Whether you're configuring containers, defining CI/CD pipelines, or setting up application configurations, YAML's clean syntax makes it the go-to choice for configuration files.

## Related Topics
- [Agent OS Config.yml Structure](../agentos/agent-os-config-yml-structure.md)