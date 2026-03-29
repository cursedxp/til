# OpenClaw Core Concepts: Skills, Agents & Automation

## What is OpenClaw?

OpenClaw is a personal AI assistant platform that runs on your own infrastructure. It connects to messaging channels (Telegram, Discord, etc.) and gives your AI access to tools, skills, and a persistent workspace with memory.

---

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
- They're loaded by the agent on demand
- Think of them as: "instructions for using a tool"

### Agents
An **agent** is an independent AI brain. Each agent has its **own isolated workspace** — completely separate from other agents or the main workspace.

```
/home/suzi/.openclaw/workspace/   ← agent's own home
├── SOUL.md        # Who the agent is (personality, values)
├── USER.md        # Who the human is (name, preferences, context)
├── IDENTITY.md    # Agent's name, avatar, emoji
├── MEMORY.md      # Long-term curated memory (only loaded in main session)
├── AGENTS.md      # Operational rules and conventions
├── TOOLS.md       # Environment-specific notes (SSH, devices, etc.)
├── HEARTBEAT.md   # What to check on periodic polls
└── memory/        # Daily raw session logs
    └── 2026-03-29.md
```

**Key difference:** Skills = knowledge, Agents = identity + memory + autonomy

Each agent persists across sessions via these files. Without them, it wakes up fresh every time with no memory.

---

## Main Workspace Structure

The workspace the agent manages for the *human* is separate:

```
workspace/
├── projects/      # One .md per project
├── documents/     # Research, notes, drafts, references
└── operations/    # Automations, scripts, configs (one folder per automation)
```

Agents live *outside* this structure — they use files from `operations/` but maintain their own workspace.

---

## Memory Architecture

| File | Purpose | When Loaded |
|------|---------|-------------|
| `MEMORY.md` | Curated long-term memory | Main session only (security) |
| `memory/YYYY-MM-DD.md` | Daily raw session logs | Current + previous day |
| `USER.md` | Human context | Every session |
| `SOUL.md` | Agent personality | Every session |

> **Why not always load MEMORY.md?** It contains personal context that shouldn't leak in group chats or shared sessions.

---

## Automation Methods

| Method | When to Use |
|--------|-------------|
| **Heartbeat** | Periodic checks that can batch together (email + calendar in one turn) |
| **Cron** | Exact timing matters, isolated one-shot tasks |
| **Shell script** | Triggered or scripted automations |
| **PingFyr** | Scheduled delivery across 6 channels (Email, Webhook, Slack, Discord, Telegram, OpenClaw) |

### Heartbeat vs Cron
- **Heartbeat** → agent checks `HEARTBEAT.md` on each poll, batches multiple tasks
- **Cron** → exact schedule, runs in isolation, direct channel output

---

## Key Insight

> OpenClaw is not just a chatbot wrapper — it's a framework for giving AI **persistent identity**, **memory**, and **autonomous action** while keeping humans in control. Each agent has its own home, its own soul, and its own memory — it's less of a tool and more of a teammate.
