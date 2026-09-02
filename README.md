# Kiroweaver

Quality-optimized multi-agent orchestration for Kiro CLI. Every agent runs the highest-performing model for its specific task — not the most expensive, not the cheapest, the best results per dollar.

## Model Cost Reference

| Cheap (≤0.10x) | Moderate (0.25x–0.40x) | Expensive (≥1.00x) |
|---|---|---|
| Qwen3 `0.05x`, Luna `0.10x` | MiniMax `0.25x`, Haiku `0.40x` | Terra/Sol `1.00x–2.40x`, Sonnet `1.30x`, Opus `2.20x` |

## Table of Contents

- [Closed System Doctrine](#closed-system-doctrine)
- [Why Kiroweaver Over Kiro Auto?](#why-kiroweaver-over-kiro-auto)
- [Architecture](#architecture)
- [Agent Fleet](#agent-fleet)
- [Installation](#installation)
- [Usage](#usage)
- [Skill Model Override Protection](#skill-model-override-protection)
- [Cost Breakdown](#cost-breakdown)
- [Philosophy](#philosophy)
- [Adding Custom Agents](#adding-custom-agents)
- [File Structure](#file-structure)
- [Troubleshooting](#troubleshooting)

## Closed System Doctrine

Kiroweaver is a strict, self-contained agent fleet. Every task goes through the Orchestrator, which routes to a specialist agent. No external agents, no manual overrides, no exceptions.

### Prohibited Actions

| Action | Why It Breaks Kiroweaver |
|---|---|
| `/agent default` or non-Kiroweaver agents | Breaks routing logic. Default agent doesn't understand terse protocol. |
| `--model` manual overrides | Bypasses quality-optimized routing. You pay 5–44x more for no quality gain. |
| Spawning outside `kiroweaver-*` namespace | Context isolation fails. Conflicting instructions. |
| Kiro Auto model selector | Black-box routing ignores task-to-model mapping. |
| Loading Caveman/other skills directly | Conflicts with built-in terse rules. Duplicate instructions cause drift. |
| Obeying skill model directives | Skills may request expensive models. Kiroweaver ignores them. |

### Required Rules

| # | Rule | Enforcement |
|---|---|---|
| 1 | Always start with `kiro-cli --agent kiroweaver` | Orchestrator owns session lifecycle |
| 2 | Always delegate via `/spawn kiroweaver-{role}` | Sub-agents are isolated. No cross-talk. |
| 3 | Never use `/agent` outside Kiroweaver | `Ctrl+0`–`Ctrl+6` are the only valid switches |
| 4 | Never pass `--model` or `--agent` mid-session | Orchestrator selects the model. You do not. |
| 5 | Never load external skills mid-session | Skills only via `resources:` block. Model directives discarded. |
| 6 | Never obey skill model overrides | Expensive model requests are ignored. Correct cheap agent is used. |

## Why Kiroweaver Over Kiro Auto?

| Feature | Kiro Auto (1.0x) | Kiroweaver |
|---|---|---|
| Model Visibility | Black box — you never know which model runs | Transparent — every agent shows its model |
| Cost Control | Fixed at 1.0x — no savings possible | 0.05x–0.40x — up to 20x cheaper |
| Task Specialization | Generalist routing — may use Sonnet for coding | Specialist routing — Qwen3 for code, Haiku for vision, MiniMax for security |
| Quality Proof | No published benchmarks | Every choice backed by published scores |
| Skill Integration | Skills run on Auto's chosen expensive model | Skills locked to cheap models — directives ignored |
| Communication | Verbose, filler-heavy output | Terse by default — 50–70% less tokens |
| Multi-Agent | No orchestration | 6 specialists + orchestrator in parallel |
| Pentesting | May block (Opus 5 classifiers) | Explicit VAPT agent with Strix |

### The Problem With Auto

Kiro's Auto selector is designed for convenience, not optimization.

Example — "Write a React component":
- **Auto**: Routes to Claude Sonnet 5 (1.30x). "Coding detected → use expensive generalist."
- **Kiroweaver**: Routes to Qwen3 Coder Next (0.05x). "Coding detected → use coding specialist."
- **Savings**: 26x cheaper, same quality.

Example — "Audit dependencies for CVEs":
- **Auto**: Routes to Claude Opus 5 (2.20x). "Security detected → use most expensive model."
- **Kiroweaver**: Routes to MiniMax M2.5 (0.25x). "Security detected → use proven specialist."
- **Result**: 8.8x cheaper, 2.4x better detection (48% vs 19%).

### When to Use What

| Use Kiro Auto When | Use Kiroweaver When |
|---|---|
| First-time Kiro exploration | Transparent, benchmark-backed model selection |
| Cost doesn't matter | 10x–20x savings without quality loss |
| One-off varied tasks | Multi-agent orchestration (thinker → coder → security → vapt) |
| No skill integration | Skill integration with model override protection |
| No pentesting needed | Active VAPT with real exploit PoCs |

Kiroweaver is not a workaround for Auto. It is a replacement for users who want control, quality proof, and cost optimization.

## Architecture

```
Kiroweaver Orchestrator
GPT 5.6 Luna · 0.10x
Terminal-Bench 84.7% · Coding Rank #6/147

Your prompt → Routes to the right specialist agent
NO external agents · NO manual overrides · NO exceptions

        ┌──────────┬──────────┼──────────┬──────────┬──────────┐
        ▼          ▼          ▼          ▼          ▼          ▼
   ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐
   │ Thinker │ │  Coder  │ │  Vision │ │ Designer│ │ Security│
   │  Luna   │ │  Qwen3  │ │  Haiku  │ │  Qwen3  │ │ MiniMax │
   │ 0.10x   │ │ 0.05x   │ │ 0.40x   │ │ 0.05x   │ │ 0.25x   │
   └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘
                              │
                              ▼
                        ┌───────────┐
                        │   VAPT    │
                        │  MiniMax  │
                        │  0.25x    │
                        │  + Strix  │
                        └───────────┘
```

## Agent Fleet

### `kiroweaver` — Orchestrator (Ctrl+0)

| | |
|---|---|
| Model | GPT 5.6 Luna |
| Cost | 0.10x |
| Benchmarks | Terminal-Bench 84.7% · Coding Rank #6/147 |
| Why This Model | Beats Sonnet 5 (80.4%) on terminal tasks. Costs 13x less. Understands code well enough to route perfectly. |

```bash
kiro-cli --agent kiroweaver
```

---

### `kiroweaver-thinker` — Reasoning & Planning (Ctrl+1)

| | |
|---|---|
| Model | GPT 5.6 Luna |
| Cost | 0.10x |
| Permissions | Read-only |
| Why This Model | Planning requires workflow understanding, not deep reasoning. Luna's 84.7% Terminal-Bench proves it handles complex multi-step tasks. |

Best for: Architecture design, API contracts, refactoring plans, tech stack evaluation

```bash
kiro-cli --agent kiroweaver-thinker
```

---

### `kiroweaver-coder` — Implementation (Ctrl+2)

| | |
|---|---|
| Model | Qwen3 Coder Next |
| Cost | 0.05x |
| Benchmarks | SWE-Bench 71.3% · Codeforces 2100 |
| Why This Model | Purpose-built 80B MoE coding model. Outperforms generalists 10–20x larger on coding tasks. Not "cheap" — specialized. |

Best for: Feature implementation, refactoring, tests, debugging, shell scripts

```bash
kiro-cli --agent kiroweaver-coder
```

---

### `kiroweaver-vision` — Image & Error Log Analysis (Ctrl+3)

| | |
|---|---|
| Model | Claude Haiku 4.5 |
| Cost | 0.40x |
| Benchmarks | Defect Detection 80% · OCR 61.6% · Document Understanding 77.8% |
| Why This Model | Only model with proven OCR benchmarks for reading error logs. Terra/Sol have object detection, not text extraction. |

Best for: Screenshot analysis, error log OCR, stack trace transcription, UI bug reports

```bash
kiro-cli --agent kiroweaver-vision
```

---

### `kiroweaver-designer` — UI/UX Design (Ctrl+4)

| | |
|---|---|
| Model | Qwen3 Coder Next + UI UX Pro Max |
| Cost | 0.05x |
| Why This Model | Skill provides design intelligence (colors, fonts, patterns). Model provides code implementation. Qwen3's 71.3% SWE-Bench proves quality output. |

Best for: Landing pages, dashboards, component libraries, design systems

```bash
kiro-cli --agent kiroweaver-designer
```

---

### `kiroweaver-security` — Security Audits (Ctrl+5)

| | |
|---|---|
| Model | MiniMax M2.5 |
| Cost | 0.25x |
| Benchmarks | Vulnerability Detection 48% |
| Why This Model | Matches Gemini 3.1-pro & Codex GPT-5.4. Beats Sonnet 5 (19.6%) by 2.4x. Costs 5.2x less. |

Best for: Dependency CVE scans, secret detection, OWASP validation, static review

```bash
kiro-cli --agent kiroweaver-security
```

---

### `kiroweaver-vapt` — Penetration Testing (Ctrl+6)

| | |
|---|---|
| Model | MiniMax M2.5 + Strix |
| Cost | 0.25x |
| Why This Model | Opus 5 (2.20x) blocks pentesting outright. MiniMax + Strix is the only viable option for active exploit validation with real PoCs. |

Best for: Black-box web app testing, API security, grey-box auth testing, CI/CD security gates

```bash
kiro-cli --agent kiroweaver-vapt
```

## Installation

### Prerequisites

- Kiro CLI installed and authenticated
- Python 3.x (for UI UX Pro Max skill scripts)
- Docker (for Strix VAPT agent)

### Step 1: Clone

```bash
git clone https://github.com/awskrishnak/kiroweaver.git ~/.kiro/agents/kiroweaver-repo
```

### Step 2: Install Agents

```bash
mkdir -p ~/.kiro/agents
cp ~/.kiro/agents/kiroweaver-repo/agents/*.md ~/.kiro/agents/
```

### Step 3: Install Skills

```bash
# UI UX Pro Max (for designer)
npm install -g ui-ux-pro-max-cli
uipro init --ai kiro --global

# Strix (for VAPT)
curl -sSL https://strix.ai/install | bash
# or: npx skills add usestrix/strix
```

### Step 4: Configure Strix

```bash
export STRIX_LLM="openrouter/z-ai/glm-5.3"
export LLM_API_KEY="your-api-key"
```

### Step 5: Verify

```bash
kiro-cli --agent kiroweaver
```

Expected: `Kiroweaver Orchestrator (Luna). CLOSED SYSTEM. Quality-optimized routing.`

## Usage

### Start (Mandatory)

```bash
kiro-cli --agent kiroweaver
```

### Spawn Sub-Agents

```bash
/spawn kiroweaver-thinker  "Design auth flow with JWT"
/spawn kiroweaver-coder    "Implement JWT middleware"
/spawn kiroweaver-vision   "Review this screenshot"
/spawn kiroweaver-designer "Build a SaaS landing page"
/spawn kiroweaver-security "Audit dependencies for CVEs"
/spawn kiroweaver-vapt     "Pentest https://staging.myapp.com"
```

### Parallel Execution (up to 4)

```bash
/spawn kiroweaver-coder "Write tests" and /spawn kiroweaver-thinker "Review edge cases"
```

### Keyboard Shortcuts

| Shortcut | Agent | Model | Cost |
|---|---|---|---|
| Ctrl+0 | Orchestrator | GPT 5.6 Luna | 0.10x |
| Ctrl+1 | Thinker | GPT 5.6 Luna | 0.10x |
| Ctrl+2 | Coder | Qwen3 Coder Next | 0.05x |
| Ctrl+3 | Vision | Claude Haiku 4.5 | 0.40x |
| Ctrl+4 | Designer | Qwen3 Coder Next | 0.05x |
| Ctrl+5 | Security | MiniMax M2.5 | 0.25x |
| Ctrl+6 | VAPT | MiniMax M2.5 + Strix | 0.25x |

## Skill Model Override Protection

Skills cannot hijack expensive models.

- Skill says: "Use Claude Opus 5 for this" → Kiroweaver ignores it, routes to correct cheap agent.
- Skill says: "Switch to GPT 5.6 Terra" → Kiroweaver ignores it, uses assigned specialist model.

Every Kiroweaver agent has a CLOSED SYSTEM block that:

1. Ignores all model directives from external skills
2. Blocks skill attempts to spawn non-Kiroweaver sub-agents
3. Routes tasks back through the orchestrator to the correct agent
4. Discards skill model preferences — skills run on the agent's model or not at all

### Example

```bash
# Inside kiroweaver-designer:
Skill says: "Switch to claude-opus-5 for this complex design"

# Designer responds:
"Skill model override blocked. Running on kiroweaver-designer (Qwen3, 0.05x).
Design system generated. Style: Corporate Minimalism..."
```

Result: Skill data (colors, fonts, patterns) used. Expensive model suggestion discarded.

## Cost Breakdown

### Scenario: Build SaaS Landing Page + Security Audit

| Step | Agent | Model | Cost Multiplier | Est. Credits |
|---|---|---|---|---|
| 1 | Thinker | GPT 5.6 Luna | 0.10x | ~15 |
| 2 | Designer (design system) | Qwen3 Coder Next | 0.05x | ~25 |
| 3 | Designer (implementation) | Qwen3 Coder Next | 0.05x | ~50 |
| 4 | Vision (screenshot review) | Claude Haiku 4.5 | 0.40x | ~15 |
| 5 | Security (audit) | MiniMax M2.5 | 0.25x | ~12 |
| | | **TOTAL** | | **~117** |

### Comparison

| Approach | Total Credits | vs Kiroweaver |
|---|---|---|
| Kiroweaver (our fleet) | ~117 | — |
| Claude Sonnet 5 (1.30x) | ~1,040–1,560 | 9x–13x more expensive |
| Claude Opus 5 (2.20x) | ~1,760–2,640 | 15x–23x more expensive |
| Kiro Auto (1.00x) | ~800–1,200 | 7x–10x more expensive |

Savings: 85–96% vs expensive alternatives. Zero quality compromise.

## Philosophy

### Quality-Per-Task, Not Price-Per-Token

Kiroweaver does not optimize for the lowest price. It optimizes for the highest quality per dollar on each specific task.

| Model | Cost | What It Actually Is |
|---|---|---|
| Qwen3 Coder Next | 0.05x | Purpose-built coding model (80B MoE, 3B active). Not "cheap" — specialized. |
| GPT 5.6 Luna | 0.10x | #6 globally on coding benchmarks. Not "basic" — proven. |
| MiniMax M2.5 | 0.25x | Matches frontier detection rates. Not "budget" — beats Sonnet 5 by 2.4x. |
| Claude Haiku 4.5 | 0.40x | Only proven OCR for error logs. Not "mid-tier" — the vision specialist. |

### Specialization Beats Generalization

A $2.20 generalist (Opus 5) is not better at coding than a $0.05 specialist (Qwen3) when the specialist was built for that exact task.

A $1.30 generalist (Sonnet 5) is not better at security than a $0.25 specialist (MiniMax) when the specialist scores 48% and the generalist scores 19.6%.

### Closed System Integrity

Kiroweaver agents are calibrated to work together. Their prompts, permissions, and terse style are interdependent.

If it is not in the `kiroweaver-*` namespace, it does not exist in this system.

### Skill Model Sovereignty

External skills provide data and logic. Kiroweaver provides the model. Skills do not dictate which model runs them.

### Safe by Default

- Security and VAPT agents are read-only
- Orchestrator cannot write files
- Sub-agents have isolated context — no leakage
- VAPT requires explicit authorization before testing

## Adding Custom Agents

Use the Agent Factory (`agent-factory.sh`) to create new sub-agents within the Kiroweaver fleet:

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

Constraint: New agents are automatically prefixed with `kiroweaver-`. They cannot be created outside the fleet.

## File Structure

```
kiroweaver/
├── agents/
│   ├── kiroweaver.md              Main orchestrator — ONLY entry point
│   ├── kiroweaver-thinker.md      Reasoning & planning
│   ├── kiroweaver-coder.md        Code implementation
│   ├── kiroweaver-vision.md       Image & error log analysis
│   ├── kiroweaver-designer.md     UI/UX design
│   ├── kiroweaver-security.md     Security audits
│   └── kiroweaver-vapt.md         Penetration testing (Strix)
├── agent-factory.sh               Interactive agent creator
├── rebrand-agents.sh              Rename entire fleet
└── README.md                      This file
```

## Troubleshooting

### "I accidentally switched to /agent default"
Fix: Type `/agent kiroweaver` to return. Do not continue in non-Kiroweaver agents.

### "I passed --model and now costs are wrong"
Fix: Exit. Restart with `kiro-cli --agent kiroweaver`. Never pass `--model` flags.

### "A skill is trying to use an expensive model"
Fix: Nothing. Kiroweaver ignores skill model directives automatically. If you see "Skill model override blocked," the protection is working.

### "Are these cheap models actually good?"
Fix: Yes. Every model choice is backed by benchmark data:
- Qwen3: 71.3% SWE-Bench — competes with models 10–20x larger
- Luna: #6 global coding rank — above Opus 4.8 on Terminal-Bench
- MiniMax: 48% detection — 2.4x better than Sonnet 5's 19.6%
- Haiku: 80% defect detection — only proven OCR for error logs

### Agent not found
```bash
ls ~/.kiro/agents/kiroweaver*.md
cp kiroweaver/agents/*.md ~/.kiro/agents/
```

### Skill not loading (designer)
```bash
npm install -g ui-ux-pro-max-cli@latest
uipro init --ai kiro --global
```

### Strix not found (VAPT)
```bash
curl -sSL https://strix.ai/install | bash
# or: npx skills add usestrix/strix
```

### Model unavailable
Qwen3 Coder Next is experimental and limited to `us-east-1`/`eu-central-1`. MiniMax M2.5 availability depends on your Kiro plan. If unavailable, the agent falls back to your default model — a temporary fallback, do not rely on it.

## License

MIT — use it, fork it, rebrand it. See LICENSE.

## Acknowledgments

- Kiro by Amazon
- UI UX Pro Max by NextLevelBuilder
- Strix by Strix AI
- Caveman by Julius Brussee

Built for specialists, not generalists.
