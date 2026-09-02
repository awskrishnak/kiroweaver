# Kiroweaver

> A multi-agent orchestration system for [Kiro CLI](https://kiro.dev). Route tasks to specialized AI agents — each running the optimal model for the job.

[![Kiro](https://img.shields.io/badge/Built%20for-Kiro%20CLI-purple)](https://kiro.dev)
[![Agents](https://img.shields.io/badge/Agents-5-blue)](./agents/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

---

## Why Kiroweaver?

Most AI coding assistants use **one model for everything**. Kiroweaver uses **five specialized agents**, each tuned to a specific task and running the cheapest capable model:

| Task | Typical Approach | Kiroweaver Approach | Savings |
|---|---|---|---|
| Planning | Claude Sonnet (1.0x) | **Luna** (0.1x) | **10×** |
| Coding | Claude Sonnet (1.0x) | **Qwen3 Coder** (0.05x) | **20×** |
| Design | Claude Sonnet (1.0x) | **Qwen3 Coder** (0.05x) | **20×** |
| Image review | Claude Sonnet (1.0x) | **Terra** (1.0x) | Same |
| Security audit | Claude Sonnet (1.0x) | **Terra** (1.0x) | Same |

**Result:** You pay pennies for what others pay dollars for — without sacrificing quality.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    KIROWEAVER (Orchestrator)                 │
│                     GPT-5.6 Luna · 0.1x                      │
│                                                              │
│  Your prompt → Routes to the right specialist agent          │
└─────────────────────────────────────────────────────────────┘
                              │
        ┌──────────┬──────────┼──────────┬──────────┐
        ▼          ▼          ▼          ▼          ▼
   ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐
   │ Thinker │ │  Coder  │ │  Vision │ │ Designer│ │ Security│
   │  Luna   │ │  Qwen3  │ │  Terra  │ │  Qwen3  │ │  Terra  │
   │  0.1x   │ │ 0.05x   │ │ 1.0x    │ │ 0.05x   │ │ 1.0x    │
   └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘
```

---

## Agents

### 1. `kiroweaver` — Orchestrator
**Model:** GPT-5.6 Luna (0.1x)

The brain of the system. Does not write code or analyze images. It **reads your intent and routes** to the right specialist. Think of it as a project manager that never does the work itself — it just delegates perfectly.

**When to use:** Start every session with this agent. It handles mixed tasks by spawning sub-agents in parallel.

```bash
kiro-cli --agent kiroweaver
```

**Keyboard shortcut:** `Ctrl+0`

---

### 2. `kiroweaver-thinker` — Reasoning & Planning
**Model:** GPT-5.6 Luna (0.1x)

Handles architecture, system design, tradeoff analysis, and task sequencing. **Does not generate code.** If you ask it to code, it will route you to the coder instead.

**Best for:**
- Designing auth flows
- Database schema decisions
- API contract design
- Refactoring plans
- Tech stack evaluation

```bash
kiro-cli --agent kiroweaver-thinker
```

**Keyboard shortcut:** `Ctrl+1`

---

### 3. `kiroweaver-coder` — Implementation
**Model:** Qwen3 Coder Next (0.05x)

Purpose-built for code generation. Reads files, writes code, runs tests, and debugs. Uses tools aggressively. The cheapest way to write production code in Kiro.

**Best for:**
- Writing features end-to-end
- Refactoring existing code
- Writing unit/integration tests
- Debugging errors
- Shell scripting

```bash
kiro-cli --agent kiroweaver-coder
```

**Keyboard shortcut:** `Ctrl+2`

---

### 4. `kiroweaver-vision` — Image Analysis
**Model:** GPT-5.6 Terra (1.0x)

Analyzes screenshots, UI mockups, diagrams, and visual bug reports. Describes exactly what it sees and flags issues.

**Best for:**
- UI bug reports from screenshots
- Design review
- Accessibility issues
- Diagram interpretation
- Visual regression analysis

```bash
kiro-cli --agent kiroweaver-vision
```

**Keyboard shortcut:** `Ctrl+3`

---

### 5. `kiroweaver-designer` — UI/UX Design
**Model:** Qwen3 Coder Next (0.05x) + [UI UX Pro Max](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill)

Generates complete design systems (colors, typography, spacing, patterns) and implements them in your stack. Integrates with the UI UX Pro Max skill for industry-specific design intelligence.

**Best for:**
- Landing pages
- Dashboards
- Component libraries
- Design system generation
- Responsive layouts

```bash
kiro-cli --agent kiroweaver-designer
```

**Keyboard shortcut:** `Ctrl+4`

---

### 6. `kiroweaver-security` — Security Audits
**Model:** GPT-5.6 Terra (1.0x)

Read-only security analysis. Scans dependencies, reviews code for vulnerabilities, checks for secrets, and flags insecure patterns. **Never modifies files.**

**Best for:**
- Dependency vulnerability scans
- Secret detection
- Insecure pattern review
- OWASP checklist validation
- Pre-commit security review

```bash
kiro-cli --agent kiroweaver-security
```

**Keyboard shortcut:** `Ctrl+5`

---

## Installation

### Prerequisites
- [Kiro CLI](https://kiro.dev/docs/) installed and authenticated
- Python 3.x (for UI UX Pro Max skill scripts)

### Step 1: Clone the agents

```bash
git clone https://github.com/yourname/kiroweaver.git ~/.kiro/agents/kiroweaver-repo
```

### Step 2: Copy agent configs to Kiro

```bash
mkdir -p ~/.kiro/agents
cp ~/.kiro/agents/kiroweaver-repo/agents/*.md ~/.kiro/agents/
```

### Step 3: Install UI UX Pro Max skill (optional, for designer)

```bash
npm install -g ui-ux-pro-max-cli
uipro init --ai kiro --global
```

### Step 4: Verify

```bash
kiro-cli --agent kiroweaver
```

You should see: `🪨 Kiroweaver Orchestrator (Luna). Routing: thinker | coder | vision | designer | security.`

---

## Usage

### Start with the orchestrator
```bash
kiro-cli --agent kiroweaver
```

### Switch agents mid-session
```bash
/agent kiroweaver-coder
/agent kiroweaver-designer
```

### Spawn sub-agents from the orchestrator
```bash
# Single task
/spawn kiroweaver-thinker "Design the auth flow"
/spawn kiroweaver-coder "Implement JWT middleware"
/spawn kiroweaver-security "Audit dependencies for CVEs"

# Parallel (up to 4)
/spawn kiroweaver-coder "Write tests" and /spawn kiroweaver-thinker "Review edge cases"
```

### Use keyboard shortcuts
| Shortcut | Agent |
|---|---|
| `Ctrl+0` | Orchestrator |
| `Ctrl+1` | Thinker |
| `Ctrl+2` | Coder |
| `Ctrl+3` | Vision |
| `Ctrl+4` | Designer |
| `Ctrl+5` | Security |

---

## Cost Breakdown

### Example: Build a SaaS landing page

| Step | Agent | Model | Credits |
|---|---|---|---|
| Plan the page | Thinker | Luna (0.1x) | ~15 |
| Generate design system | Designer | Qwen3 (0.05x) | ~25 |
| Implement the code | Designer | Qwen3 (0.05x) | ~50 |
| Review screenshot | Vision | Terra (1.0x) | ~30 |
| Audit for secrets | Security | Terra (1.0x) | ~20 |
| **Total** | | | **~140** |

**Same task on Claude Sonnet (1.0x):** ~800–1200 credits.

**Savings: ~85–90%**

---

## Philosophy

### Terse by default
All Kiroweaver agents use a terse communication style:
- Drop articles, filler words, and hedging
- Fragments are OK
- Code blocks are exact — never abbreviated
- Auto-clarity: if you're confused, say "explain" and the agent switches to full clarity mode

### Model-per-task, not model-per-session
Why pay 1.0x for planning when Luna (0.1x) is faster and good enough? Why use a generalist for coding when Qwen3 Coder (0.05x) was built for it? Kiroweaver routes every task to the right tool.

### Safe by default
- Security agent is read-only
- Orchestrator cannot write files
- Sub-agents have isolated context — they don't leak into each other

---

## Adding Custom Agents

Use the [Agent Factory](./agent-factory.sh) script to create new sub-agents:

```bash
chmod +x agent-factory.sh
./agent-factory.sh
```

The script will:
1. Detect your orchestrator
2. Ask for agent specs (name, model, permissions, skills)
3. Generate the agent `.md` file
4. Auto-update the orchestrator's routing table
5. Assign the next free keyboard shortcut

---

## File Structure

```
kiroweaver/
├── agents/
│   ├── kiroweaver.md              # Main orchestrator
│   ├── kiroweaver-thinker.md      # Reasoning & planning
│   ├── kiroweaver-coder.md        # Code implementation
│   ├── kiroweaver-vision.md       # Image analysis
│   ├── kiroweaver-designer.md     # UI/UX design
│   └── kiroweaver-security.md     # Security audits
├── agent-factory.sh               # Interactive agent creator
├── rebrand-agents.sh              # Rename all agents
└── README.md                      # This file
```

---

## Requirements

- Kiro CLI 1.x or later
- Valid Kiro authentication
- Python 3.x (for UI UX Pro Max skill, optional)
- Node.js + npm (for UI UX Pro Max CLI, optional)

---

## Troubleshooting

### Agent not found
```bash
# Verify the file exists
ls ~/.kiro/agents/kiroweaver*.md

# If missing, re-copy from the repo
cp kiroweaver/agents/*.md ~/.kiro/agents/
```

### Skill not loading (designer)
```bash
# Re-install UI UX Pro Max
npm install -g ui-ux-pro-max-cli@latest
uipro init --ai kiro --global
```

### Model unavailable
Qwen3 Coder Next is experimental and limited to `us-east-1` and `eu-central-1` regions. If unavailable, the agent falls back to your default model.

---

## License

MIT — use it, fork it, rebrand it. See [LICENSE](./LICENSE).

---

## Acknowledgments

- [Kiro](https://kiro.dev) by Amazon — the agentic IDE that makes this possible
- [UI UX Pro Max](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) by NextLevelBuilder — design intelligence for the designer agent
- [Caveman](https://github.com/JuliusBrussee/caveman) by Julius Brussee — inspiration for the terse communication style
