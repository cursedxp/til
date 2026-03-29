# OpenClaw Core Concepts: Skills, Agents & Automation

## What is OpenClaw?

OpenClaw is a personal AI assistant platform that runs on your own infrastructure. It connects to messaging channels (Telegram, Discord, etc.) and gives your AI access to tools, skills, and your personal workspace.

## Skills vs Agents

### Skills
A **skill** is a knowledge package — a `SKILL.md` file that tells the AI *how* to use a specific tool or service.

```
skills/
├── github/SKILL.md       # How to use gh CLI
├── weather/SKILL.md      # How to fetch weather
└── obsidian/SKILL.md     # How to work with Obsidian
```

- Skills don't run independently
- They're loaded by the agent when needed
- Think of them as: "instructions for using a tool"

### Agents
An **agent** is an independent AI brain with its own:
- Workspace (files, memory)
- Personality (`SOUL.md`)
- Long-term memory (`MEMORY.md`)
- Context about the user (`USER.md`)

```
workspace/
├── SOUL.md        # Who the agent is
├── USER.md        # Who the human is
├── MEMORY.md      # Long-term curated memory
└── memory/        # Daily raw notes
    └── 2026-03-29.md
```

**Key difference:** Skills = knowledge, Agents = identity + memory + autonomy

## Workspace Structure

```
workspace/
├── projects/      # One .md per project
├── documents/     # Research, notes, drafts
└── operations/    # Automations, scripts, configs
```

Agents live *outside* this structure — they use files from `operations/` but have their own workspace.

## Automation Methods

| Method | When to Use |
|--------|-------------|
| **Heartbeat** | Periodic checks that can batch together (email + calendar in one turn) |
| **Cron** | Exact timing matters, isolated tasks |
| **Shell script** | One-shot or triggered automations |
| **PingFyr** | Scheduled delivery across 6 channels (Email, Webhook, Slack, Discord, Telegram, OpenClaw) |

## Key Insight

> OpenClaw is not just a chatbot wrapper — it's a framework for giving AI persistent identity, memory, and autonomous action while keeping humans in control.
