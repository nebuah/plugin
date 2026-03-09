# Nebuah

**A Bio-Inspired Cognitive Operating System for Legal Research, Literature & Legal Consulting**

> *"What if your AI legal assistant remembered what worked, learned from what failed, and got smarter every time you used it?"*

Nebuah transforms Claude from a **stateless assistant that forgets everything between sessions** into a **learning system** that accumulates strategies, avoids past mistakes, and improves with every interaction. It brings the algorithmic rigor of hierarchical memory consolidation — proven in physical robotics — to the Claude Code Desktop plugin ecosystem, specialized for the legal domain.

Nebuah uses **unihemispheric dreaming** — inspired by dolphins, which sleep with one brain hemisphere while the other stays alert. Instead of waiting for a nightly consolidation cycle, Nebuah can dream *while you work*: parallel dream sessions consolidate traces in the background, focused on specific goals or broad sweeps, on-demand or on a schedule.

---

## Table of Contents

- [The Problem](#the-problem)
- [The Solution](#the-solution-nebuah)
- [What Nebuah Adds to Claude Code Desktop](#what-nebuah-adds-to-claude-code-desktop)
- [Architecture Overview](#architecture-overview)
- [Installation](#installation)
- [Unihemispheric Dreaming](#unihemispheric-dreaming)
  - [Loop Mode — Recurring Dreams with `/loop`](#loop-mode--recurring-dreams-with-loop)
- [Tutorial: Using Nebuah](#tutorial-using-nebuah)
- [Directory Structure](#directory-structure)
- [How the Dream Engine Works](#how-the-dream-engine-works)
- [Seed Strategies](#seed-strategies)
- [Strategy Format Reference](#strategy-format-reference)
- [Provenance](#provenance)
- [License](#license)

---

## The Problem

AI legal assistants today suffer from **cognitive amnesia**. Every time you start a new session with Claude, the slate is wiped clean:

**1. No memory of what worked.** You developed a thorough contract review approach last week using a specific pattern for flagging non-standard indemnification clauses. Today, facing a similar review, Claude starts from scratch — suggesting generic approaches that miss the nuances you already identified.

**2. No memory of what failed.** Last month, Claude cited a case that had been overruled. Today, facing a similar research task, it might make the exact same mistake — there's no persistent record of that lesson.

**3. No accumulation of expertise.** You've been working on M&A due diligence for months. Claude has helped you review dozens of contracts. But it still doesn't "know" your firm's review patterns — it doesn't remember that you always check for change-of-control provisions, that your client requires specific indemnification caps, or that you use a particular clause hierarchy in your analysis memos.

**4. No strategic planning.** Claude treats every legal task as if it's the first one. It doesn't decompose complex engagements into hierarchical sub-tasks, doesn't track which analyses depend on others, and doesn't learn from the outcomes of multi-step legal work.

**5. Repeated mistakes across sessions.** Without persistent memory, the same anti-patterns reappear. The same research gaps get overlooked. The same time gets wasted.

### The Cost of Forgetting

| Scenario | Without Nebuah | With Nebuah |
|----------|----------------|--------------|
| Reviewing vendor contracts | Claude uses a generic approach every time | Claude applies `strat_3_document-review` with your firm's specific checklist |
| Researching case law | Claude might cite overruled cases | `_negative_constraints.md` blocks citing without verification; `seed_2_legal-research` enforces systematic checking |
| Drafting a new contract | Claude guesses your conventions | `seed_3_contract-drafting` follows your established clause structure |
| Second week on a case | Same quality as day one | Strategies evolved from 10+ successful traces guide better analysis |

---

## The Solution: Nebuah

Nebuah is a **Claude Code Desktop plugin** that implements a biologically-inspired cognitive architecture. It is based on three innovations, originally proven in [RoClaw](https://github.com/EvolvingAgentsLabs/RoClaw) — a physical robot that learns to navigate rooms by consolidating motor traces into reusable strategies during "sleep" phases.

### 1. Hierarchical Task Decomposition (L1-L4)

Every task is decomposed into a 4-level cognitive hierarchy, mirroring neocortical organization:

| Level | Name | Legal Equivalent | Example |
|-------|------|---------------------|---------|
| **L1** | GOAL | Engagement / Case | "Handle corporate M&A due diligence" |
| **L2** | ARCHITECTURE | Strategy / Framework | "Design regulatory compliance framework" |
| **L3** | TACTICAL | Document / Task | "Draft non-disclosure agreement" |
| **L4** | REACTIVE | Action / Query | `Search case law on indemnification caps` |

Traces at each level link to their parent, forming a tree that the Dream Engine can analyze for patterns.

### 2. Execution Traces (Short-Term Memory)

Every significant action is logged as a structured trace with: hierarchy level, parent linkage, goal, applied strategy, outcome (SUCCESS/FAILURE/PARTIAL), confidence score, and action details. These traces are the raw material for learning.

### 3. Unihemispheric Dream Engine (3-Phase Memory Consolidation)

Dolphins never fully sleep — one brain hemisphere consolidates memories while the other remains active. Nebuah works the same way. Dream sessions run **in parallel with active work**, triggered on-demand, by goal, or on a schedule. Each dream is focused: it can process all traces, or only traces matching specific goals or hierarchy levels.

Each dream session executes the same 3-phase cycle:

| Phase | Biological Analog | What It Does |
|-------|-------------------|-------------|
| **Phase 1: SWS** | Slow Wave Sleep | Replays failures. Extracts **negative constraints** — things to NEVER do again. |
| **Phase 2: REM** | REM Sleep | Abstracts successful patterns into **reusable strategies** with confidence scores. Merges new evidence into existing strategies. |
| **Phase 3: Consolidation** | Memory Writing | Writes strategies and constraints to disk. Updates the dream journal. Prunes old traces. |

---

## What Nebuah Adds to Claude Code Desktop

| Capability | Claude Code Desktop Alone | Claude Code Desktop + Nebuah |
|-----------|--------------------------|-------------------------------|
| **Memory** | `CLAUDE.md` + auto-memory (preferences only) | Hierarchical strategies with confidence scores, negative constraints, dream journal |
| **Learning** | None between sessions | Dream Engine extracts strategies from successes and constraints from failures |
| **Unihemispheric Dreaming** | Not possible | Dream sessions run in parallel with work — on-demand, goal-focused, scheduled, or all three at once |
| **Planning** | Ad-hoc, flat task decomposition | 4-level hierarchical decomposition (L1-L4) with strategy injection |
| **Error prevention** | None | Negative constraints loaded before every task; high-severity constraints always enforced |
| **Task logging** | No structured logging | Hierarchical traces with parent-child linking, outcomes, and confidence |
| **Knowledge reuse** | Manual copy-paste | Automatic strategy matching via trigger keywords and composite scoring |

---

## Architecture Overview

| RoClaw (Robotics) | Nebuah (Legal) |
|---|---|
| Execution traces (bytecodes) | Session logs (tool calls, research, document ops) |
| Motor hierarchy (L1 Goal → L4 Motor) | Task hierarchy (L1 Engagement → L4 Action) |
| Negative constraints (collision warnings) | Anti-patterns (don't cite overruled cases, don't skip jurisdiction check) |
| Strategy (doorway approach pattern) | Strategy (contract review workflow, legal research pattern) |
| Cortex (goal planner) | SystemAgent |
| Hippocampus (memory consolidation) | DreamEngineAgent |
| Circadian sleep cycle | **Replaced**: Unihemispheric (dolphin) dreaming — parallel, continuous, goal-focused |

### Three Core Agents

| Agent | Biological Role | Function |
|-------|----------------|----------|
| **SystemAgent** | Cortex | Executive orchestrator — decomposes goals, queries strategies, delegates to specialized agents |
| **MemoryAnalysisAgent** | Hippocampal Encoder | Real-time trace logger — captures execution events as structured hierarchical traces |
| **DreamEngineAgent** | Hippocampus | Memory consolidator — runs the 3-phase SWS/REM/Consolidation cycle. Supports parallel, goal-focused dreaming. |

---

## Installation

### Prerequisites

- **Claude Pro, Max, Team, or Enterprise** subscription
- **Claude Desktop** app: Download from [claude.com/download](https://claude.com/download) (macOS or Windows)

### Step 1: Install the Plugin

Open **Claude Desktop** and switch to the **Code** tab. Then install the Nebuah plugin using one of these methods:

#### Method A: Install via the Plugin Manager (Recommended)

1. Click the **+** button next to the prompt box
2. Select **Plugins**
3. Select **Add plugin** to open the plugin browser
4. Search for `nebuah` and install it

#### Method B: Ask Claude to Install It

In any Code session, simply ask:

```
Please install the nebuah plugin globally
```

### Step 2: Open Your Project

1. Start a **new session** in the Code tab
2. Select your project folder — or create a new one
3. Nebuah activates automatically when the plugin is installed

The `system/memory/` directory will be created in your project folder on first use.

### Step 3: Verify Installation

After installation, test that Nebuah is active:

```
/nebuah List all available seed strategies
```

Claude should report 6 bootstrap strategies covering case intake, legal research, contract drafting, regulatory compliance, document review, and case law analysis.

---

## Unihemispheric Dreaming

Dolphins sleep with one brain hemisphere at a time — they never fully stop. Nebuah works the same way: **dream sessions run in parallel with active work sessions**.

| Trigger | How It Works | Best For |
|---------|-------------|----------|
| **On-demand** | `/nebuah dream [keywords]` | Consolidate after finishing a specific research task or document review |
| **After task completion** | Automatic — `/nebuah` triggers per-agent dream cycles (min 3) | Every task contributes to learning immediately |
| **Loop mode** | `/nebuah loop` generates a `/loop` command for recurring dreams | Active working sessions — lightweight, session-scoped |
| **Scheduled** | Recurring Desktop task (hourly, daily, etc.) | Full-sweep catch-all to consolidate anything the focused dreams missed |
| **Parallel multi-goal** | `/nebuah dream --parallel contracts \| compliance \| litigation` | Dream about multiple domains simultaneously after a big sprint |

### On-Demand Dreaming

```
/nebuah dream                                  # Full sweep — process all unprocessed traces
/nebuah dream contract review                  # Goal-focused — only contract-related traces
/nebuah dream L3                               # Level-focused — only tactical traces
/nebuah dream regulatory compliance L3         # Combined — compliance tactical traces
/nebuah dream --parallel contracts | lit | reg  # Parallel — three dream sessions at once
/nebuah dream status                           # Check memory state, unprocessed trace count
```

### Loop Mode — Recurring Dreams with `/loop`

```
/nebuah loop                              # Outputs: /loop 1h /nebuah dream
/nebuah loop contract review              # Outputs: /loop 1h /nebuah dream contract review
/nebuah loop 30m regulatory compliance    # Outputs: /loop 30m /nebuah dream regulatory compliance
/nebuah loop stop                         # Outputs instructions for stopping
```

---

## Tutorial: Using Nebuah

### Level 1: Passive Mode (Zero Effort)

Once installed, Nebuah works **automatically** through `CLAUDE.md` instructions. You don't need to learn any new commands.

### Level 2: Active Mode — Using `/nebuah`

```
/nebuah Review the vendor contracts in contracts/vendor/ for non-standard indemnification terms
```

**What Nebuah does:**
1. Queries strategies → finds `seed_3_document-review`
2. Checks constraints → loads seed constraints (verify citations, check jurisdiction)
3. Follows the document review strategy adapted to your specific task
4. Logs L3 TACTICAL traces with actions and outcomes
5. Runs per-agent dream consolidation
6. Next time: higher-confidence strategy, even better execution

### Level 3: Complex Multi-Step Engagement

```
/nebuah Handle complete due diligence for the proposed acquisition of TargetCo
```

**What Nebuah does:**
1. Creates an L1 GOAL trace: "Handle M&A due diligence"
2. Decomposes into L2 sub-tasks: contract review, regulatory compliance, litigation exposure, IP analysis
3. Creates specialized agents for each workstream (minimum 3)
4. Executes with trace logging at every level
5. After completion, runs parallel dream cycles — one per agent
6. Reports: strategies applied, new strategies learned, constraints discovered

---

## Directory Structure

```
nebuah/
├── .claude/
│   └── agents/                       # Agent definitions for Claude Code
│       ├── SystemAgent.md            # Cortex: orchestration & planning
│       ├── MemoryAnalysisAgent.md    # Encoder: trace logging
│       └── DreamEngineAgent.md       # Hippocampus: 3-phase consolidation
├── .claude-plugin/
│   └── marketplace.json              # Marketplace config
├── nebuah/                           # Plugin package
│   ├── .claude-plugin/
│   │   └── plugin.json               # Plugin manifest (v3.0.0)
│   ├── agents/                       # Same 3 agents (plugin scope)
│   ├── commands/
│   │   └── nebuah.md                 # /nebuah kernel command
│   └── system_files/                 # Reference specifications
│       ├── ClaudeCodeToolMap.md
│       ├── MemoryTraceManager.md
│       ├── QueryMemoryTool.md
│       └── SmartMemory.md
├── system/
│   └── memory/
│       ├── strategies/                # Long-term memory (persistent)
│       │   ├── level_1_epics/
│       │   ├── level_2_architecture/
│       │   ├── level_3_tactical/
│       │   ├── level_4_reactive/
│       │   ├── _seeds/                # Bootstrap strategies (6 legal seeds)
│       │   ├── _negative_constraints.md
│       │   └── _dream_journal.md
│       └── traces/                    # Short-term memory (volatile, gitignored)
├── CLAUDE.md                          # Kernel instructions
├── .gitignore
├── LICENSE                            # Apache 2.0
└── README.md                          # This file
```

---

## How the Dream Engine Works

### Phase 1: Slow Wave Sleep (SWS) — Failure Analysis

Replays failures and extracts negative constraints (anti-patterns). Example:

```markdown
### Never cite a case without verifying subsequent history
**Context:** legal research, case law analysis
**Severity:** high
**Learned from:** tr_1709736622000_a3f2
```

### Phase 2: REM Sleep — Strategy Abstraction

Abstracts successful patterns into reusable strategies. Merges new evidence into existing strategies.

### Phase 3: Consolidation — Persistence

Writes strategies and constraints to disk, updates the dream journal, prunes old traces.

### Strategy Scoring Algorithm

```
composite = (trigger_match * 0.5) + (confidence * 0.3) + (success_rate * 0.2)
```

Strategies with composite score < 0.2 are filtered out. Top 5 matches are returned.

---

## Seed Strategies

Nebuah ships with 6 bootstrap strategies for the legal domain:

| ID | Level | Title | Trigger Keywords |
|----|-------|-------|-----------------|
| `seed_1_case-intake` | L1 | New Case/Engagement Intake | new case, client intake, matter opening |
| `seed_2_legal-research` | L2 | Legal Research Workflow | legal research, case law, find precedent |
| `seed_3_contract-drafting` | L3 | Contract/Agreement Drafting | draft contract, NDA, agreement |
| `seed_3_regulatory-compliance` | L3 | Regulatory Compliance Review | regulatory, compliance audit, licensing |
| `seed_3_document-review` | L3 | Legal Document Review & Analysis | document review, due diligence, clause analysis |
| `seed_4_case-law-analysis` | L4 | Case Law Analysis Strategy | analyze case, brief a case, case precedent |

Seeds start with `confidence: 0.5` and evolve through use.

---

## Strategy Format Reference

```yaml
---
id: strat_3_contract-review          # Unique ID: strat_[level]_[slug]
version: 2                           # Incremented by Dream Engine on merge
hierarchy_level: 3                   # 1=GOAL, 2=ARCH, 3=TACTICAL, 4=REACTIVE
title: Contract Review Pattern       # Human-readable name
trigger_goals:                       # Keywords for matching
  - contract review
  - due diligence
  - clause analysis
preconditions:                       # What must be true to apply
  - documents available for review
confidence: 0.78                     # 0.0-0.95, updated by Dream Engine
success_count: 5
failure_count: 1
source_traces:
  - tr_1709736622000_a3f2
deprecated: false
---

# Contract Review Pattern

## Steps
1. Define review scope and create checklist
2. Organize documents by category
3. Review each document against checklist
4. Flag non-standard terms and risk areas
5. Cross-reference related documents
6. Compile risk-rated findings memo

## Negative Constraints
- Never skip cross-referencing between related documents
- Never present findings without severity ratings

## Notes
- For large document sets, use keyword searching to prioritize
```

---

## Provenance

Nebuah stands on the shoulders of two projects:

- **[RoClaw](https://github.com/EvolvingAgentsLabs/RoClaw)**: The hierarchical memory architecture, Dream Engine 3-phase algorithm, strategy scoring, negative constraints, trace hierarchy (L1-L4), and the core+adapter pattern.

- **[LLMunix Marketplace](https://github.com/EvolvingAgentsLabs/llmunix-marketplace)**: The pure-markdown plugin architecture, agent definitions with YAML frontmatter, the kernel command, SmartMemory system, and the philosophy that everything should be inspectable markdown.

---

## License

Apache License 2.0 — See [LICENSE](LICENSE)

Built by Nebuah Labs
