# Claude Code Tool Map — Nebuah Edition

Maps Nebuah cognitive operations to Claude Code's native tool inventory. Updated for the hierarchical trace system and strategy-driven execution model in legal research, literature, and legal consulting.

## Core Tool Inventory

### File Operations
| Tool | Purpose | Trace Level |
|------|---------|-------------|
| `Read` | Read file contents (contracts, briefs, statutes) | L4 |
| `Write` | Create/overwrite files (drafts, memos, analyses) | L3-L4 |
| `Edit` | Precise string replacements (clause edits, corrections) | L3-L4 |
| `Glob` | Find files by pattern (locate documents) | L4 |
| `NotebookEdit` | Edit Jupyter notebooks | L3-L4 |

### Search Operations
| Tool | Purpose | Trace Level |
|------|---------|-------------|
| `Grep` | Search file contents by regex (find clauses, terms) | L4 |
| `WebSearch` | Search the web (case law, regulations, legal databases) | L2-L3 |
| `WebFetch` | Fetch and process web content (legal sources) | L2-L3 |

### Execution Operations
| Tool | Purpose | Trace Level |
|------|---------|-------------|
| `Bash` | Run shell commands (document processing, file ops) | L3-L4 |
| `Task` | Delegate to sub-agents (legal specialists) | L1-L2 |

### Google Drive Operations (MCP)
| Tool | Purpose | Trace Level |
|------|---------|-------------|
| `mcp__claude_ai_Google_Drive__create_file` | Upload files to GDrive (base64 content), create GDrive folders and native docs | L3-L4 |
| `mcp__claude_ai_Google_Drive__download_file_content` | Download raw file content from GDrive | L3-L4 |
| `mcp__claude_ai_Google_Drive__read_file_content` | Read natural language representation of GDrive files | L4 |
| `mcp__claude_ai_Google_Drive__search_files` | Search GDrive with structured queries (title, fullText, mimeType, parentId) | L4 |
| `mcp__claude_ai_Google_Drive__list_recent_files` | List recently modified files | L4 |
| `mcp__claude_ai_Google_Drive__get_file_metadata` | Get file metadata (size, owner, dates) | L4 |
| `mcp__claude_ai_Google_Drive__get_file_permissions` | Get file sharing permissions | L4 |

## Nebuah Operation Mappings

### 1. Strategy Query
```
Operation: Find relevant strategies before planning
Tools: Grep → Read
Pattern:
  1. Grep(pattern="keyword", path="system/memory/strategies/", output_mode="files_with_matches")
  2. Read(matching_files) → parse YAML frontmatter → score → rank
```

### 2. Constraint Loading
```
Operation: Load negative constraints before execution
Tools: Read
Pattern:
  1. Read("system/memory/strategies/_negative_constraints.md")
  2. Filter by context relevance to current legal task
```

### 3. Trace Writing
```
Operation: Log execution trace
Tools: Read → Write (or Edit to append)
Pattern:
  1. Read("system/memory/traces/trace_YYYY-MM-DD.md")  # Check if exists
  2. Write/Edit to append new trace entry
```

### 4. Agent Creation
```
Operation: Create specialized agent for delegation
Tools: Write → Read → Task
Pattern:
  1. Write("projects/[case]/components/agents/[Agent].md", agent_definition)
  2. Read(agent_file)  # Load full definition
  3. Task(subagent_type="general-purpose", prompt=agent_content)
```

### 5. Dream Consolidation
```
Operation: Run 3-phase memory consolidation
Tools: Task
Pattern:
  1. Task(subagent_type="DreamEngineAgent", prompt="Run dream consolidation...")
  # DreamEngineAgent internally uses: Glob, Read, Write, Grep, Bash
```

### 6. Project Initialization
```
Operation: Create case/engagement directory structure
Tools: Bash (mkdir)
Pattern:
  1. Bash("mkdir -p projects/[case]/{components/agents,output,memory/{short_term,long_term}}")
```

### 7. Strategy Application
```
Operation: Apply a strategy during execution
Tools: Read
Pattern:
  1. Read the matching strategy file
  2. Extract steps from markdown body
  3. Follow steps in order, creating L3/L4 traces
  4. After completion, update trace with outcome
```

### 8. Strategy Writing (Dream Engine Only)
```
Operation: Write new or updated strategy
Tools: Write
Pattern:
  1. Generate YAML frontmatter + markdown body
  2. Write to system/memory/strategies/level_[N]_[name]/[slug].md
```

### 9. GDrive Input Download
```
Operation: Download project input documents from Google Drive to local
Tools: mcp__claude_ai_Google_Drive__search_files → mcp__claude_ai_Google_Drive__read_file_content → Write
Pattern:
  1. search_files(query="parentId = '[input_folder_id]'") → list of files
  2. For each file: read_file_content(fileId) → text content
  3. Write(local_path, content)
```

### 10. GDrive Output Upload
```
Operation: Upload project outputs to Google Drive
Tools: Read → mcp__claude_ai_Google_Drive__create_file
Pattern:
  1. Read(local_output_file) → content
  2. Base64 encode the content
  3. create_file(title, mimeType, parentId, content=base64)
```

### 11. GDrive Memory Sync
```
Operation: Sync strategies between local and Google Drive
Tools: Glob → Read → mcp__claude_ai_Google_Drive__search_files → mcp__claude_ai_Google_Drive__create_file
Pattern:
  1. Glob("system/memory/strategies/level_*/*.md") → local strategies
  2. search_files(query="parentId = '[gdrive_strategies_id]'") → remote strategies
  3. Compare → upload new/updated local strategies to GDrive
```

### 12. Cross-Project Memory Query
```
Operation: Search past project memories in Google Drive
Tools: mcp__claude_ai_Google_Drive__search_files → mcp__claude_ai_Google_Drive__read_file_content
Pattern:
  1. search_files(query="fullText contains '[keyword]' and parentId = '[strategies_folder_id]'")
  2. read_file_content(matching_file_ids) → strategy text
  3. Score and inject into agent context
```

## Common Workflow Patterns

### Full Task Execution
```
1. [Grep] Query strategies for relevant legal patterns
2. [Read] Load negative constraints
3. [Read] Load matching strategy details
4. [Write] Create root trace (L1/L2)
5. [Write/Task] Execute sub-tasks (creating L3/L4 traces)
6. [Write] Update root trace with outcome
7. [Task] Invoke DreamEngineAgent for consolidation
```

### Strategy-Guided Legal Work
```
1. [Grep] Find strategy matching "draft [document type]"
2. [Read] Load strategy steps
3. For each step:
   a. [Write] Create L3 trace
   b. [Read/Write/Edit/WebSearch] Execute the step
   c. [Write] Update L3 trace with outcome
4. [Write] Update L2 trace with aggregate outcome
```

### GDrive-Enhanced Task Execution
```
1. [GDrive] Search for input folder → download input documents to local
2. [Grep] Query local strategies
3. [GDrive] Search cross-project memories for additional strategies
4. [Read] Load constraints
5. [Write] Create traces, execute tasks locally
6. [GDrive] Upload outputs to project's GDrive output folder
7. [GDrive] Sync updated strategies to GDrive
8. [Task] Dream consolidation
```

### Post-Failure Learning
```
1. [Write] Log failure trace with detailed reason
2. [Task] Invoke DreamEngineAgent
3. DreamEngine Phase 1 (SWS):
   a. [Read] Parse failure traces
   b. [Write] Add new negative constraint
4. DreamEngine Phase 3:
   a. [Write] Update _negative_constraints.md
   b. [Write] Append to _dream_journal.md
```

## Tool Limitations & Workarounds

| Limitation | Workaround |
|-----------|------------|
| `Write` cannot create files in non-existent directories | Use `Bash(mkdir -p)` first |
| `Task` is one-shot (no follow-up) | Include ALL context in the initial prompt |
| `Grep` is single-line by default | Use `multiline: true` for cross-line patterns |
| `Bash` has 2-minute default timeout | Use `timeout` parameter for long operations |
| `Edit` requires unique `old_string` | Include more surrounding context for uniqueness |
| `GDrive create_file` content must be base64 encoded | Use `btoa()` or base64 encoding |
| `GDrive read_file_content` may truncate very large files | Use `download_file_content` for complete data |
| `GDrive search_files` query syntax requires exact operator format | See GDriveSync.md for query examples |
