# Nebuah Kernel

You are operating within **Nebuah**, a bio-inspired cognitive operating system for legal research, literature, and legal consulting. This system brings the algorithmic rigor of hierarchical memory consolidation (proven in robotics) to Claude Code and Claude Cowork workflows, specialized for the legal domain.

## Working Directory (read this first)

Every path in this contract — `system/memory/…`, `projects/…`,
`system/gdrive_registry.json` — is **relative to the current working
directory**, not to where the plugin is installed. Nebuah operates on the
workspace you launched Claude Code in.

A valid Nebuah workspace has a `projects/` directory and a `system/memory/`
directory. Before doing anything else:

1. Check that `projects/` and `system/memory/` exist in the CWD.
2. If they don't, **stop and ask** which workspace to use. Do not create a
   Nebuah tree inside an unrelated repository.
3. To share a workspace with `nebuah-engine` (the local Python runtime), run
   Claude Code from the engine's checkout. The two write the same project
   format — see **Project Format** below.

## Core Architecture

Nebuah implements a **4-level cognitive hierarchy** adapted from neuroscience:

| Level | Name | Legal Equivalent | Example |
|-------|------|---------------------|---------|
| L1 | GOAL | Engagement / Case | "Handle corporate M&A due diligence" |
| L2 | ARCHITECTURE | Strategy / Framework | "Design regulatory compliance framework" |
| L3 | TACTICAL | Document / Task | "Draft non-disclosure agreement" |
| L4 | REACTIVE | Action / Query | `Search: case law on indemnification caps` |

## Before Starting Any Complex Task

1. **Check negative constraints** to avoid past mistakes:
   - Read `system/memory/strategies/_negative_constraints.md`
   - Apply ALL constraints with severity `high` unconditionally
   - Apply `medium` constraints when context matches

2. **Search for relevant strategies** before planning:
   - Use `Grep` on `system/memory/strategies/` for keywords related to the user's goal
   - If a strategy with `confidence >= 0.5` matches, APPLY its steps
   - If multiple strategies match, prefer the one with highest `success_count`

3. **Check the dream journal** for recent learnings:
   - Read `system/memory/strategies/_dream_journal.md` for the last 3 entries

4. **Auto-bootstrap Google Drive** (first run only, then no-op):
   - If `system/gdrive_registry.json` doesn't exist: search GDrive for "Nebuah" folder → use/create → build system sub-folders → write registry
   - Only ask the user if multiple root folders found (disambiguation)
   - After bootstrap, all GDrive operations are fully automatic

5. **Search cross-project memories in Google Drive** (automatic):
   - Use `mcp__<gdrive>__search_files` with `fullText contains '[goal keywords]'`
   - Load matching strategies with `mcp__<gdrive>__read_file_content`
   - Add as Priority 15 context (between constraints and local strategies)

## During Task Execution

### Trace Logging
All significant actions MUST be logged as traces. Use the following format for trace files in `system/memory/traces/`:

**File naming**: `trace_YYYY-MM-DD.md` (one file per day, append new traces)

**Trace format**:
```markdown
### Time: [ISO 8601]
**Trace ID:** tr_[timestamp]_[random]
**Level:** [1-4]
**Parent:** [parent_trace_id or null]
**Goal:** [what was being attempted]
**Strategy:** [strategy_id if one was applied, or null]
**Outcome:** [SUCCESS | FAILURE | PARTIAL | ABORTED | UNKNOWN]
**Reason:** [brief explanation of outcome]
**Duration:** [time in ms if measurable]
**Confidence:** [0.0-1.0]

**Actions:**
1. [Tool: action description] -> [result summary]
2. [Tool: action description] -> [result summary]
---
```

### Hierarchy Rules
- L1 traces wrap entire case/engagement executions
- L2 traces wrap strategic decisions and framework design phases
- L3 traces wrap individual document/analysis implementations
- L4 traces wrap individual actions and queries
- Every L2+ trace MUST reference a parent trace ID

## After Completing a Complex Task

Invoke the `DreamEngineAgent` sub-agent using the `Task` tool to consolidate learnings:

```
Task(subagent_type="nebuah:DreamEngineAgent", prompt="Run dream consolidation cycle on recent traces in system/memory/traces/")
```

This triggers the 3-phase bio-inspired consolidation:
1. **SWS (Slow Wave Sleep)**: Analyze failures, extract negative constraints
2. **REM Sleep**: Abstract successful patterns into reusable strategies
3. **Consolidation**: Write strategies to disk, update journal, prune old traces

## Project Format (shared with nebuah-engine)

Nebuah Engine and this plugin write **the same project folder**. The engine's
format is canonical; this plugin conforms to it. The full spec is
`docs/PROJECT_FORMAT.md` in the engine repo.

```
projects/<nucleus>/<slug>/          # nucleus: legal | literary | _inbox
├── project.json                    # descriptor: state, phase, plan (below)
├── decisions.json                  # open questions for the client
├── components/
│   ├── agents/                     # one .md per agent + ROSTER.md + PROVENANCE.md
│   └── skills/<slug>/SKILL.md      # methodologies this project actually used
├── input/  knowledge/  methods/
├── memory/{short_term,long_term/{agent_templates,domain_knowledge,workflow_patterns}}
├── output/{sections,assets,governance}
└── traces/                         # per project, tree by initiating agent
    ├── _index.jsonl                # trace_id → path
    └── <agent>/<ts>_<agent>_<task>.md
```

### Rules that make the handoff safe

1. **`schema_version`**. `project.json` carries an integer. Absent means 1.
   If you find a version higher than 1, refuse to write — do not guess.
2. **Never drop a key you don't understand.** When you rewrite
   `project.json`, preserve every field that was already there. The engine
   does the same for yours. A silently dropped key is a silently lost
   project.
3. **Traces go in the project**, as `traces/<agent>/<ts>_<agent>_<task>.md`
   with YAML frontmatter, and every trace is appended to `traces/_index.jsonl`.
   The old global `system/memory/traces/trace_YYYY-MM-DD.md` is legacy.
4. **`hierarchy_level` is optional** in a trace. When absent it is inferred
   from the depth of the `parent_trace_id` chain. Say so when you infer it.

### `project.json` — the fields you must respect

| Field | Note |
|---|---|
| `id` | `proj_<YYYYMMDD>_<slug>` |
| `state` | `proposed` \| `active` \| `completed` \| `archived` |
| `schema_version` | integer; absent = 1 |
| `phase` | see the phase machine below |
| `phase_data.plan_tasks[]` | the plan: `id`, `title`, `skill`, `output_file`, `depends_on`, `worker_instruction`, `mode`, `priority` |
| `confidential` | when true, **nothing leaves the machine** — no GDrive, no external calls |

### Phase machine

```
intake → knowledge_base → planning → awaiting_educator → awaiting_team
       → executing → reviewing → delivering → awaiting_feedback
       → iterating → completed
                              ↘ blocked
```

Empty `phase` means `intake`. **Resuming a project means reading `phase` and
`phase_data.plan_tasks`, not starting over.** Execute the tasks that have no
output yet, in `depends_on` order.

### `components/agents/ROSTER.md`

The team, bound to tasks **by name** — never by position. YAML frontmatter with
`project_id`, `nucleus`, `version`, `approved_by`, `changelog`, and `members[]`:
`name`, `task`, `mode` (`spawn`/`evolve`/`reuse`), `derived_from`,
`derived_sha`, `model`, `exam_score`, `provisional`, `file`.

`model` is a **logical role**, not a vendor model id: `razonamiento-largo`,
`redaccion`, `verificacion`. Each runtime maps it to what it has — the engine
to `ollama/gemma4:26b`, this plugin to the Claude model in session. Writing
`gemini-3.5-flash` into a roster makes it unportable.

## Taking a Project (cooperative lock)

`nebuah-engine` runs a tick loop over these same project folders. If you both
advance a project at once, `phase_data` ends up in a state neither of you
wrote.

**Before writing `phase` or `phase_data`:**

```
1. Read project.json. If `locked_by` is set and `locked_at` is less than
   30 minutes old and it isn't you → do NOT write state. Say who holds it.
2. Otherwise set:  locked_by: "claude-code",  locked_at: <ISO 8601 UTC>
3. Do the work.
4. Clear both fields when you finish — including when you fail.
```

Two things that are not optional:

* **A lock expires.** 30 minutes, `NEBUAH_LOCK_TTL_MINUTES` in the engine. A
  process that dies must not freeze an engagement forever. If you find an
  expired lock, take it and say so.
* **Only clear your own.** If `locked_by` names someone else, leave it.

You may always read, and always write to `output/`, `traces/` and
`components/agents/` — those never conflict. The lock is only for `phase` and
`phase_data`.

## Memory Architecture

```
system/memory/
├── strategies/               # Long-term memory (persistent knowledge)
│   ├── level_1_epics/        # Engagement-level patterns  ← L1 is the HIGHEST
│   ├── level_2_architecture/ # Legal strategy & framework strategies
│   ├── level_3_tactical/     # Document/task-level tactics
│   ├── level_4_reactive/     # Action-level patterns      ← L4 is the LOWEST
│   ├── _seeds/               # Bootstrap strategies (never deleted)
│   ├── _negative_constraints.md  # What NOT to do
│   └── _dream_journal.md    # Consolidation history
├── stores/hybrid-memory/     # The engine's own store (domain-keyed, no frontmatter)
│   ├── strategies/<domain>/
│   └── constraints/<domain>/
└── gdrive_registry.json      # Google Drive folder ID mappings
```

**The level convention is load-bearing: L1 is the highest.** Until 2026-09-02
the engine used the same `level_N_` prefix with the height inverted
(`level_1_reflexive` was the lowest), so writing across systems filed every
strategy at the wrong level, silently. The engine's directories were empty and
were renamed to match this convention. Never reintroduce the old names.

The engine's `stores/hybrid-memory/` uses a different shape (plain markdown,
keyed by domain, no counters). Translate with `services/memory_adapter.py` in
the engine repo — never read or write it by hand.

### Google Drive — one tree, shared with the engine

```
Nebuah/
├── proyectos/<project_id>/     # Spanish: the engine got here first and the
│   ├── LEEME.md                #   client is already looking at these folders
│   ├── bitacora.md             # full chronology of the engagement
│   ├── 00_recibido/            # what the client sent
│   ├── 01_progreso/            # daily progress, one .md per day
│   ├── 02_entregables/         # deliverables
│   └── _sistema/               # project.json + components/ — how to RESUME
├── _bandeja_entrada/<sender>/
├── _respaldos/<project_id>/<ts>.zip
└── system/memory/strategies/level_*/
```

There used to be two trees under the same root — `projects/` written by this
plugin over the Drive API, and `proyectos/` written by the engine into a
Drive-for-Desktop folder. They shared the root and nothing else. **Write to
`proyectos/`.** `_sistema/` is what makes a project resumable from Drive; the
client never needs to open it.

Never mirror `traces/` to Drive: raw execution detail, client data.

## Strategy Format

Strategies are Markdown files with YAML frontmatter:

```yaml
---
id: strat_[level]_[slug]
version: 1
hierarchy_level: [1-4]
title: [Human-readable title]
trigger_goals: ["keyword1", "keyword2"]
preconditions: ["condition1"]
confidence: 0.5
success_count: 0
failure_count: 0
source_traces: []
deprecated: false
---
```

## Claude Cowork Integration

When running in **Claude Cowork** (background agents):
- The **SystemAgent** (Cortex) handles user-facing orchestration
- The **DreamEngineAgent** (Hippocampus) can run asynchronously in background to consolidate memory while you work
- The **MemoryAnalysisAgent** handles real-time trace logging

## Execution Modes

### Default Multi-Agent Mode (Triad Decomposition)

Every `/nebuah [goal]` invocation **always creates at least 3 agents** along these concern axes:

| Agent Role | Responsibility |
|---|---|
| **Research Agent** | Core legal research (case law, statutes, doctrinal analysis) |
| **Quality Agent** | Compliance review, citation verification, factual accuracy |
| **Integration Agent** | Document formatting, deliverable assembly, client-ready output |

For complex goals, additional agents are created beyond the minimum 3. After execution, **one dream cycle per agent** runs in parallel (minimum 3), each filtered by that agent's keywords.

### System Control Mode (Sysctl)

`/nebuah sysctl [mode] [project]` invokes the system control plane for project maintenance. Delegates to skillos-systemcontrol-plugin agents:

- `/nebuah sysctl audit [project]` → Security scan of agent definitions
- `/nebuah sysctl score [project]` → Performance scoring with 7-type failure taxonomy
- `/nebuah sysctl evolve [project]` → Controlled improvements with anti-overfitting gate
- `/nebuah sysctl prune [project]` → Dead agent detection and memory compaction
- `/nebuah sysctl full [project]` → All modes in sequence
- `/nebuah sysctl health` → Health report across all projects

All sysctl reports are uploaded to GDrive. See `nebuah/system_files/SysctlProtocol.md` for the full specification.

### Loop Mode (Recurring Dreams)

`/nebuah loop` generates `/loop` commands for the user to copy-paste (the plugin cannot invoke `/loop` directly since it's a built-in CLI command):

- `/nebuah loop` → outputs: `/loop 1h /nebuah dream`
- `/nebuah loop [keywords]` → outputs: `/loop 1h /nebuah dream [keywords]`
- `/nebuah loop [interval] [keywords]` → outputs: `/loop [interval] /nebuah dream [keywords]`
- `/nebuah loop stop` → outputs cancellation instructions

Loop commands are session-scoped (max 3 days) and stop when the session ends or Ctrl+C is pressed.

## Google Drive Integration (MANDATORY — Never Optional)

Nebuah integrates with Google Drive as a **MANDATORY requirement**. Every file produced by the system MUST be saved to Google Drive. The user types `/nebuah [goal]` and all GDrive operations happen transparently — no setup commands, no manual sync. **GDrive is NOT optional** — if the MCP tools are available, they MUST be used.

### Enforcement Rules
- **NEVER silently skip** GDrive uploads. If `system/gdrive_registry.json` is missing, bootstrap it immediately — do not skip.
- **ALWAYS use `disableConversionToGoogleType: true`** when uploading markdown, strategy, or any text file. Use `mimeType: "text/plain"` (not `"text/markdown"`). This prevents Google from auto-converting files and corrupting YAML frontmatter.
- **NEVER report task success** without confirming GDrive uploads completed. The execution report MUST include GDrive status.
- **If an upload fails**, retry once. If it still fails, log the failure explicitly — never hide it.

### How It Works
1. **Auto-bootstrap** (first run only): Detects or creates the Nebuah root folder in GDrive. Only asks the user if disambiguation is needed.
2. **Project creation**: MUST create GDrive project folders (input/, output/, memory/) and download any input documents.
3. **Output upload**: MUST upload ALL deliverables to GDrive after production. Non-optional.
4. **Memory sync**: MUST upload ALL strategies/constraints/journal to GDrive after dream consolidation. Non-optional.
5. **Cross-project memory**: Searches GDrive for strategies from past projects during planning.

### Manual Override Commands (Advanced)
- `/nebuah gdrive pull [project]` — Force download input documents
- `/nebuah gdrive push [project]` — Force upload outputs + memories
- `/nebuah gdrive sync` — Force bidirectional system memory sync
- `/nebuah gdrive status` — Show sync state

### Strategy Upload Rule
Always upload strategies with `disableConversionToGoogleType: true` to preserve YAML frontmatter.

## Reference Specifications

Detailed specifications for system internals are in `nebuah/system_files/`:
- **SmartMemory.md**: Memory hierarchy, retrieval algorithm, and context assembly
- **MemoryTraceManager.md**: Trace schema, lifecycle, and hierarchy rules
- **QueryMemoryTool.md**: Strategy scoring algorithm and query patterns
- **ClaudeCodeToolMap.md**: Tool-to-operation mapping and workflow patterns
- **GDriveSync.md**: Google Drive integration, sync protocols, cross-project memory access, and agent definition upload protocol
- **SysctlProtocol.md**: System control — security audits, performance scoring, evolution with anti-overfitting gate, lifecycle management, and memory compaction

Consult these when implementing complex memory operations, debugging the consolidation pipeline, or performing system maintenance.

## Important Rules

1. NEVER delete seed strategies in `_seeds/` - they are the bootstrap knowledge
2. ALWAYS log traces for tasks that take more than 2 tool calls
3. PREFER applying existing strategies over improvising from scratch
4. UPDATE strategy confidence after each application (success: +0.05, failure: -0.1)
5. DEPRECATE strategies where `failure_count > success_count * 2`
6. **MANDATORY**: SYNC ALL outputs and strategies to Google Drive — never skip, never silently fail. If registry is missing, bootstrap it. If upload fails, retry once and report.
7. **NEVER upload traces** to Google Drive — they carry raw execution detail and client data. **DO upload agent definitions** (`components/agents/*.md`): they are what makes a past engagement reusable, and GDrive folder permissions are the access control. A project marked `confidential: true` uploads NOTHING.
8. NEVER modify cross-project memories in GDrive (read-only access)
9. AUTO-BOOTSTRAP GDrive on first run — never require manual setup commands
10. NEVER ask the user more than ONE question about GDrive setup (only for root folder disambiguation)
11. **MANDATORY**: ALWAYS use `mimeType: "text/plain"` with `disableConversionToGoogleType: true` when uploading ANY file to GDrive. Never use `text/markdown` without this flag. Google auto-converts markdown to Google Docs and corrupts YAML frontmatter.
12. **MANDATORY**: A task is NOT complete until its outputs are confirmed uploaded to Google Drive. The execution report MUST show GDrive sync results. Zero GDrive uploads when local files exist = FAILURE.
