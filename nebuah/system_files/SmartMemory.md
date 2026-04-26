# Smart Memory — Nebuah Edition

Defines the persistent knowledge layer and memory hierarchy for the Nebuah cognitive architecture. Adapted from RoClaw's section-based memory system with a domain-agnostic adapter for legal research, literature, and legal consulting.

## Memory Architecture

```
system/memory/
├── strategies/                    # LONG-TERM MEMORY (persistent, local)
│   ├── level_1_epics/             # Engagement-level patterns
│   ├── level_2_architecture/      # Legal strategy & framework strategies
│   ├── level_3_tactical/          # Document/task-level tactics
│   ├── level_4_reactive/          # Action-level patterns
│   ├── _seeds/                    # Bootstrap strategies (immutable)
│   ├── _negative_constraints.md   # Anti-patterns & guardrails
│   └── _dream_journal.md         # Consolidation history
├── traces/                        # SHORT-TERM MEMORY (volatile, local only)
│   └── trace_YYYY-MM-DD.md        # Daily execution logs
└── gdrive_registry.json           # GDrive folder ID mappings

Google Drive (Nebuah/):             # CLOUD MEMORY (persistent, shared)
├── projects/[ProjectName]/
│   ├── input/                     # Source documents
│   ├── output/                    # Generated deliverables
│   └── memory/long_term/          # Project-specific learnings
└── system/memory/strategies/      # Mirror of local strategies
    ├── level_1_epics/
    ├── level_2_architecture/
    ├── level_3_tactical/
    └── level_4_reactive/
```

## Memory Types

### Short-Term Memory (Traces)

**Nature**: Volatile, ephemeral, raw
**Location**: `system/memory/traces/`
**Lifecycle**: Created during execution → consumed by Dream Engine → pruned after 7 days
**Properties**:
- Granular: Individual execution events with full context
- Chronological: Timestamped for sequence reconstruction
- Hierarchical: Linked via parent-child trace IDs
- Complete: Contains goals, actions, outcomes, confidence scores

### Long-Term Memory (Strategies)

**Nature**: Persistent, abstracted, reusable
**Location**: `system/memory/strategies/`
**Lifecycle**: Created by Dream Engine → queried before tasks → updated with new evidence
**Properties**:
- Abstract: Generalized patterns, not specific instances
- Hierarchical: Organized by cognitive level (L1-L4)
- Scored: Confidence and success/failure counts for reliability
- Queryable: YAML frontmatter with trigger_goals for efficient retrieval
- Evolvable: Version-tracked, merged with new evidence over time

### Seed Memory (Bootstrap Knowledge)

**Nature**: Immutable, foundational
**Location**: `system/memory/strategies/_seeds/`
**Lifecycle**: Created at system initialization → never modified or deleted
**Properties**:
- Provides cold-start knowledge for common legal research and consulting patterns
- Low initial confidence (0.5) — must be validated through use
- Serve as templates for Dream Engine to create evolved strategies

### Cloud Memory (Google Drive)

**Nature**: Persistent, shared, cross-session
**Location**: Google Drive `Nebuah/` folder tree
**Lifecycle**: Synced after dream consolidation → queried for cross-project knowledge → updated with new learnings
**Properties**:
- Shared: Accessible across machines and sessions
- Project-organized: Each project has its own input/output/memory
- Cross-project: Strategies from past projects queryable by keyword
- Format-flexible: Supports Google Docs, PDF, Markdown, plain text
- Sovereignty: Data stays in the user's Google Workspace (never on third-party servers)

**Supported Input Formats**:
| Format | GDrive MIME Type | Read Method | Local Format |
|--------|-----------------|-------------|-------------|
| Google Docs | `application/vnd.google-apps.document` | `read_file_content` | Markdown (.md) |
| PDF | `application/pdf` | `read_file_content` | Text extraction |
| Markdown | `text/markdown` | `download_file_content` | As-is (.md) |
| Plain Text | `text/plain` | `download_file_content` | As-is (.txt) |
| Google Sheets | `application/vnd.google-apps.spreadsheet` | `read_file_content` | Structured text |
| Word Documents | `application/vnd.openxmlformats-officedocument.wordprocessingml.document` | `read_file_content` | Text extraction |

## Memory Operations

| Operation | Agent | Input | Output |
|-----------|-------|-------|--------|
| **Write Trace** | MemoryAnalysisAgent | Execution event | Trace entry in daily file |
| **Read Traces** | DreamEngineAgent | Time window | Parsed trace sequences |
| **Consolidate** | DreamEngineAgent | Trace sequences | Strategies + constraints |
| **Query Strategy** | SystemAgent, Any | Goal keywords | Matching strategies sorted by score |
| **Apply Strategy** | Any executing agent | Strategy steps | Guided execution |
| **Update Strategy** | DreamEngineAgent | New evidence | Updated version, confidence |
| **Read Constraints** | SystemAgent, Any | Context | Applicable anti-patterns |
| **Write Constraint** | DreamEngineAgent | Failure analysis | New constraint entry |
| **GDrive Download** | SystemAgent | GDrive file ID | Local file |
| **GDrive Upload** | SystemAgent, DreamEngineAgent | Local file | GDrive file |
| **GDrive Search** | Any agent | Keywords, parentId | Matching file list |
| **Cross-Project Query** | Any agent | Goal keywords | Strategies from other projects |
| **Memory Sync** | SystemAgent | Local + GDrive state | Synchronized state |

## Strategy Retrieval Algorithm

When searching for strategies (adapted from RoClaw's composite scoring):

```
1. Extract keywords from the goal
2. For each strategy in level_*/:
   a. Match trigger_goals against keywords
      - Exact match: trigger_score = 1.0
      - Substring match: trigger_score = 0.7
      - Single word overlap: trigger_score = 0.4
      - No match: skip
   b. Calculate composite score:
      composite = (trigger_score * 0.5) + (confidence * 0.3) + (success_rate * 0.2)
      where success_rate = success_count / (success_count + failure_count)
   c. Filter: composite >= 0.2
3. Sort by composite score descending
4. Return top 5 matches
```

## Context Assembly

Before executing a task, the system assembles context from memory (inspired by RoClaw's `getFullContext()`):

```
Full Context Assembly:
├── [Priority 10] Negative Constraints (always loaded, local)
├── [Priority 15] Cross-Project Strategies (loaded from GDrive if relevant match found)
├── [Priority 20] Matched Strategies (loaded from local if query matches)
├── [Priority 30] Recent Dream Journal (last 3 entries, local)
├── [Priority 35] Project Input Documents (downloaded from GDrive at project init)
├── [Priority 40] Case-Specific Learnings (if in case/engagement context)
└── [Priority 50] Seed Strategies (fallback if no learned strategies)
```

## Memory Section Registration

Each memory section is defined by:

```yaml
name: string        # Section identifier
heading: string     # Display heading (e.g., "## Negative Constraints")
source: string      # File path or glob pattern
priority: number    # Lower = loaded first (10, 20, 30...)
required: boolean   # Whether this section must be present
```

### Default Sections for Legal Research & Consulting

| Section | Priority | Source | Required |
|---------|----------|--------|----------|
| Constraints | 10 | `strategies/_negative_constraints.md` | Yes |
| Cross-Project Strategies | 15 | GDrive `Nebuah/system/memory/strategies/` (query-matched) | No |
| Active Strategies | 20 | `strategies/level_*/` (query-matched) | No |
| Dream Journal | 30 | `strategies/_dream_journal.md` | No |
| Project Input Documents | 35 | GDrive `Nebuah/projects/[name]/input/` | No |
| Seed Strategies | 50 | `strategies/_seeds/` | No |

## Cross-Project Memory Access

Sub-agents and skills can access memories from past projects stored in Google Drive. This enables knowledge transfer between engagements.

### Query Pattern
```
1. Use mcp__claude_ai_Google_Drive__search_files with:
   query: "fullText contains '[keyword]'"
   — Searches across all project memories in GDrive

2. Filter results by parentId to scope to specific folders:
   - strategies/: for reusable patterns
   - projects/[name]/memory/: for project-specific learnings

3. Use mcp__claude_ai_Google_Drive__read_file_content to load matches

4. Score using the standard composite scoring algorithm:
   composite = (trigger_score * 0.5) + (confidence * 0.3) + (success_rate * 0.2)

5. Inject top matches into agent context at Priority 15
```

### Access Scoping
| Agent Type | Can Access |
|-----------|-----------|
| SystemAgent | All project memories + system strategies |
| Specialized Agent | Own project memories + system strategies |
| DreamEngineAgent | All traces + all strategies (for consolidation) |

## Memory Hygiene

1. **Trace Pruning**: Traces older than 7 days are deleted during dream consolidation
2. **Strategy Deprecation**: Strategies with `failure_count > success_count * 2` (min 3 attempts) are deprecated
3. **Constraint Deduplication**: Similar constraints are merged, not duplicated
4. **Journal Trimming**: Dream journal entries older than 30 days can be summarized into a single "epoch summary"
5. **Seed Preservation**: Files in `_seeds/` are NEVER modified or deleted
6. **GDrive Sync**: After every dream consolidation, sync new/updated strategies to Google Drive. Cross-project memories are read-only from GDrive (never modified remotely).
