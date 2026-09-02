# Kiroweaver

> A self-contained, cost-optimized multi-agent orchestration system for [Kiro CLI](https://kiro.dev). **Strictly uses its own agent fleet** — no external agents, no fallback models, no skill-driven model overrides, no exceptions.

[![Kiro](https://img.shields.io/badge/Built%20for-Kiro%20CLI-purple)](https://kiro.dev)
[![Agents](https://img.shields.io/badge/Agents-6-blue)](./agents/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

---

## ⚠️ CRITICAL: Kiroweaver Is a Closed System

**Do NOT use other agents, models, or skills alongside Kiroweaver.**

Kiroweaver is architected as a **strict, self-contained agent fleet**. Every task must flow through the orchestrator (`kiroweaver`) and be delegated to one of its five sub-agents. Using external agents, default models, manual model switches, or skill-driven model overrides **breaks cost optimization, voids consistency guarantees, corrupts the terse communication contract, and wastes credits.**

### What Is Prohibited

| Prohibited Action | Why It Breaks Kiroweaver |
|---|---|
| Switching to `/agent default` or any non-Kiroweaver agent | Breaks routing logic. Default agent does not understand terse protocol. |
| Manually overriding the model with `--model` flags | Bypasses cost-optimized routing. You pay 1.0x instead of 0.05x. |
| Spawning agents outside the `kiroweaver-*` namespace | Context isolation fails. Sub-agents may receive conflicting instructions. |
| Using Kiro's `Auto` model selector | Black-box routing ignores Kiroweaver's cost-aware task-to-model mapping. |
| Loading Caveman skill or other skills directly | Conflicts with Kiroweaver's built-in terse rules. Duplicate instructions cause drift. |
| Obeying model directives from external skills/plugins | Skills may request expensive models (1.0x). Kiroweaver ignores them and routes to the correct cheap agent instead. |

### What Is Required

| Rule | Enforcement |
|---|---|
| **Always start with** `kiro-cli --agent kiroweaver` | Orchestrator owns session lifecycle. |
| **Always delegate via** `/spawn kiroweaver-{role}` | Sub-agents are isolated. No cross-talk. |
| **Never use** `/agent` to switch outside Kiroweaver | Keyboard shortcuts (`Ctrl+0` through `Ctrl+5`) are the only valid switches. |
| **Never pass** `--model` or `--agent` mid-session | The orchestrator selects the model. You do not. |
| **Never load external skills** into Kiroweaver sessions | If a skill is needed, it is declared in the agent config's `resources:` block only. The skill runs on the agent's assigned model — its model directives are discarded. |
| **Never obey skill model overrides** | If a skill says "use claude-sonnet-4," Kiroweaver ignores it and routes to the correct sub-agent at 0.05x–0.1x. |

### Violation Example

```bash
# ❌ WRONG: Bypasses orchestrator, pays 1.0x, loses terse style
kiro-cli --agent default --model claude-sonnet-4
> /caveman
> Write me a React component

# ❌ WRONG: Manual model override destroys cost routing
kiro-cli --agent kiroweaver-coder --model gpt-5.6-terra

# ❌ WRONG: Spawning non-Kiroweaver agent breaks context isolation
/spawn some-other-agent "Do this"

# ❌ WRONG: Skill requests expensive model — Kiroweaver must ignore this
# (Skill says: "Switch to claude-sonnet-4 for this complex design")
```

```bash
# ✅ CORRECT: Orchestrator routes, cheapest model auto-selected
kiro-cli --agent kiroweaver
> Build a landing page
# → Spawns kiroweaver-designer (Qwen3, 0.05x) automatically

# ✅ CORRECT: Explicit spawn within fleet
/spawn kiroweaver-security "Audit dependencies"

# ✅ CORRECT: Keyboard switch within fleet
Ctrl+2  # → kiroweaver-coder

# ✅ CORRECT: Skill runs on kiroweaver-designer's model (Qwen3, 0.05x)
# Skill's request for "claude-sonnet-4" is IGNORED
```

---

## Why Kiroweaver?

Most AI coding assistants use **one model for everything**. Kiroweaver uses **six specialized agents**, each tuned to a specific task and running the cheapest capable model:

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
┌─────────────────────────────────────────────────────────────────┐
│                    KIROWEAVER (Orchestrator)                     │
│                     GPT-5.6 Luna · 0.1x                          │
│                                                                  │
│  Your prompt → Routes to the right specialist agent            │
│  NO external agents. NO manual model overrides.                │
│  NO skill-driven model switches.                               │
└─────────────────────────────────────────────────────────────────┘
                              │
        ┌──────────┬──────────┼──────────┬──────────┬──────────┐
        ▼          ▼          ▼          ▼          ▼          ▼
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
**Shortcut:** `Ctrl+0`

The brain of the system. Does not write code or analyze images. It **reads your intent and routes** to the right specialist. Think of it as a project manager that never does the work itself — it just delegates perfectly.

**Strict rule:** This is the **only** entry point. Every session starts here.

```bash
kiro-cli --agent kiroweaver
```

**Skill Model Override Protocol:** If any external skill or plugin attempts to specify its own model ("use claude-sonnet-4", "switch to gpt-5.6-terra"), the orchestrator **ignores it completely**, determines the actual task, and routes to the appropriate Kiroweaver sub-agent with the correct cheap model.

---

### 2. `kiroweaver-thinker` — Reasoning & Planning
**Model:** GPT-5.6 Luna (0.1x)  
**Shortcut:** `Ctrl+1`

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

---

### 3. `kiroweaver-coder` — Implementation
**Model:** Qwen3 Coder Next (0.05x)  
**Shortcut:** `Ctrl+2`

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

---

### 4. `kiroweaver-vision` — Image Analysis
**Model:** GPT-5.6 Terra (1.0x)  
**Shortcut:** `Ctrl+3`

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

---

### 5. `kiroweaver-designer` — UI/UX Design
**Model:** Qwen3 Coder Next (0.05x) + [UI UX Pro Max](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill)  
**Shortcut:** `Ctrl+4`

Generates complete design systems (colors, typography, spacing, patterns) and implements them in your stack. Integrates with the UI UX Pro Max skill for industry-specific design intelligence.

**Critical:** The UI UX Pro Max skill may contain model directives (e.g., "use claude-sonnet-4 for complex designs"). **These are ignored.** The designer runs ALL skill operations on Qwen3 (0.05x). The skill's data (colors, fonts, styles) is valid. Its model directives are garbage.

**Best for:**
- Landing pages
- Dashboards
- Component libraries
- Design system generation
- Responsive layouts

```bash
kiro-cli --agent kiroweaver-designer
```

---

### 6. `kiroweaver-security` — Security Audits
**Model:** GPT-5.6 Terra (1.0x)  
**Shortcut:** `Ctrl+5`

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

You should see: `🪨 Kiroweaver Orchestrator (Luna). CLOSED SYSTEM. Routing: thinker | coder | vision | designer | security.`

---

## Usage

### Start with the orchestrator (mandatory)
```bash
kiro-cli --agent kiroweaver
```

### Switch agents mid-session (Kiroweaver fleet only)
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

### Keyboard shortcuts (Kiroweaver fleet only)
| Shortcut | Agent |
|---|---|
| `Ctrl+0` | Orchestrator |
| `Ctrl+1` | Thinker |
| `Ctrl+2` | Coder |
| `Ctrl+3` | Vision |
| `Ctrl+4` | Designer |
| `Ctrl+5` | Security |

---

## Skill Model Override Protection

Kiroweaver agents are **immune to skill-driven model switches.** Here's how:

### The Problem
Many skills and plugins contain model directives:
- *"For complex tasks, switch to claude-sonnet-4"*
- *"Use gpt-5.6-terra for image analysis"*
- *"Spawn a sub-agent with Auto model selection"*

If obeyed, these directives destroy Kiroweaver's cost routing and force expensive 1.0x models.

### The Solution
Every Kiroweaver agent has a **CLOSED SYSTEM** block that:
1. **Ignores** all model directives from external skills
2. **Blocks** skill attempts to spawn sub-agents with non-Kiroweaver models
3. **Routes** the actual task back through the orchestrator to the correct cheap agent
4. **Discards** skill model preferences — skills run on the agent's assigned model or not at all

### Example
```bash
# Inside kiroweaver-designer, UI UX Pro Max skill activates:
Skill says: "Switch to claude-sonnet-4 for this complex design"

# Designer responds:
"Skill model override blocked. Running on kiroweaver-designer (Qwen3, 0.05x).
Design system generated. Style: Corporate Minimalism..."
```

**Result:** The skill's data (colors, fonts, patterns) is used. Its expensive model suggestion is thrown away. Credits saved.

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

### Closed system integrity
Kiroweaver agents are calibrated to work together. Their prompts, permissions, and terse style are interdependent. Injecting external agents, models, or skill directives breaks this calibration. **If it is not in the `kiroweaver-*` namespace, it does not exist in this system.**

### Skill model sovereignty
External skills do not dictate which model runs them. Kiroweaver agents decide. Skills provide data and logic. Kiroweaver provides the model. This separation is what makes the cost savings possible.

### Safe by default
- Security agent is read-only
- Orchestrator cannot write files
- Sub-agents have isolated context — they don't leak into each other

---

## Adding Custom Agents (Within Kiroweaver Only)

Use the [Agent Factory](./agent-factory.sh) script to create new sub-agents **within the Kiroweaver fleet**:

```bash
chmod +x agent-factory.sh
./agent-factory.sh
```

The script will:
1. Detect the Kiroweaver orchestrator
2. Ask for agent specs (name, model, permissions, skills)
3. Generate the agent `.md` file
4. Auto-update the orchestrator's routing table
5. Assign the next free keyboard shortcut

**Constraint:** New agents are automatically prefixed with `kiroweaver-`. They cannot be created outside the fleet.

---

## File Structure

```
kiroweaver/
├── agents/
│   ├── kiroweaver.md              # Main orchestrator — ONLY entry point
│   ├── kiroweaver-thinker.md      # Reasoning & planning
│   ├── kiroweaver-coder.md        # Code implementation
│   ├── kiroweaver-vision.md       # Image analysis
│   ├── kiroweaver-designer.md     # UI/UX design
│   └── kiroweaver-security.md     # Security audits
├── agent-factory.sh               # Interactive agent creator (Kiroweaver fleet only)
├── rebrand-agents.sh              # Rename entire fleet
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

### "I accidentally switched to /agent default"
**Fix:** Type `/agent kiroweaver` to return to the orchestrator. Do not continue the session in a non-Kiroweaver agent.

### "I passed --model and now costs are wrong"
**Fix:** Exit the session. Restart with `kiro-cli --agent kiroweaver`. Never pass `--model` flags.

### "A skill is trying to use an expensive model"
**Fix:** Nothing. Kiroweaver agents automatically ignore skill model directives. If you see "Skill model override blocked" in the output, the protection is working.

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
Qwen3 Coder Next is experimental and limited to `us-east-1` and `eu-central-1` regions. If unavailable, the agent falls back to your default model. This is a temporary fallback — do not rely on it.

---

## License

MIT — use it, fork it, rebrand it. See [LICENSE](./LICENSE).

---

## Acknowledgments

- [Kiro](https://kiro.dev) by Amazon — the agentic IDE that makes this possible
- [UI UX Pro Max](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) by NextLevelBuilder — design intelligence for the designer agent
- [Caveman](https://github.com/JuliusBrussee/caveman) by Julius Brussee — inspiration for the terse communication style
