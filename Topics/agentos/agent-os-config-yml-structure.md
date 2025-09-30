# Agent OS Config.yml Structure

## Overview

The Agent OS configuration file demonstrates a domain-specific YAML structure that controls Agent OS behavior and project settings.

## Configuration Structure

```yaml
agent_os_version: 1.4.1  # Version tracking

agents:                   # AI assistant configurations
  claude_code:
    enabled: false
  cursor:
    enabled: false

project_types:           # Different project configurations
  default:
    instructions: ~/.agent-os/instructions
    standards: ~/.agent-os/standards

default_project_type: default  # Active configuration
```

## Key Configuration Elements

### Version Tracking
- **agent_os_version**: Tracks which version of Agent OS configuration format is being used
- Ensures compatibility between Agent OS versions

### Agent Configuration
- **agents**: Defines which AI assistants are enabled
- Each agent can be toggled on/off with boolean flags
- Allows selective activation based on workflow needs

### Project Types
- **project_types**: Define different configuration profiles
- Each project type points to specific instruction and standards directories
- Enables different configurations for different projects

### Default Settings
- **default_project_type**: Specifies which project configuration to use
- Can be changed to switch between project configurations

## Key Observations

- **Fixed schema**: Specific keys are required by Agent OS
- **Path references**: Points to instruction directories using absolute or relative paths
- **Boolean flags**: Enable/disable specific agents
- **Extensible sections**: Can add custom project types for different workflows

## Usage Examples

### Multiple Project Types
```yaml
agent_os_version: 1.4.1

agents:
  claude_code:
    enabled: true
  cursor:
    enabled: true

project_types:
  default:
    instructions: ~/.agent-os/instructions
    standards: ~/.agent-os/standards

  web-development:
    instructions: ~/.agent-os/web-instructions
    standards: ~/.agent-os/web-standards

  data-science:
    instructions: ~/.agent-os/ds-instructions
    standards: ~/.agent-os/ds-standards

default_project_type: web-development
```

## Related Topics
- [YAML Basics and Syntax](../yaml/yaml-basics-and-syntax.md)
- [Agent OS Architecture](./agent-os_02-agent-os-architecture.md)
- [Configuration Systems Deep Dive](./agent-os_05-configuration-systems-deep-dive.md)