# Google Drive Sync — Nebuah Edition

Defines the integration layer between the Nebuah cognitive architecture and Google Drive via MCP (Model Context Protocol) tools. All project inputs, outputs, and system memories can be synchronized with a shared Google Drive folder structure, enabling cross-machine continuity, cross-project memory access, and persistent cloud-backed storage for legal research, literature, and legal consulting workflows.

## Design Principles

1. **Local-First**: All computation and file manipulation happens on the local filesystem. Google Drive is a sync target, not a working directory.
2. **Seamless Sync**: Synchronization is **fully automatic** at defined lifecycle boundaries (project creation, output production, dream consolidation). The user never needs to run explicit `gdrive` commands — the kernel handles everything. Manual `gdrive` commands exist only as overrides for advanced use.
3. **Registry-Driven**: A single local registry file maps every local path to its Google Drive folder ID. No tool call is issued without consulting the registry first.
4. **Idempotent Operations**: Running the same sync command twice produces the same state. Uploads check for existing files before creating duplicates.
5. **Zero-Config Bootstrap**: On first run, the system auto-detects or creates the Nebuah root folder in Google Drive. The user is only asked a question if disambiguation is needed (multiple root folders found). After bootstrap, all subsequent runs are fully autonomous.

## Google Drive Folder Structure

```
Nebuah/                                    # Root folder (shared)
├── projects/                              # Per-engagement workspaces
│   └── [ProjectName]/
│       ├── input/                         # Source documents (Google Docs, PDF, .txt, .md)
│       ├── output/                        # Generated deliverables
│       └── memory/
│           └── long_term/                 # Project-specific consolidated learnings
└── system/                                # Shared kernel memory
    └── memory/
        └── strategies/
            ├── level_1_epics/
            ├── level_2_architecture/
            ├── level_3_tactical/
            ├── level_4_reactive/
            ├── _negative_constraints.md
            └── _dream_journal.md
```

### Known Folder IDs

These IDs are pre-provisioned and stored in the GDrive registry:

| Path | Google Drive ID |
|------|-----------------|
| `Nebuah/` | `1Ry365y1w5yancVLmosByCnR-MWCpCmxc` |
| `Nebuah/projects/` | `1XQCMY-Hv_2pbJKgKWzyj6LMIQ23ylydl` |
| `Nebuah/system/` | `1khmm3sp7wV5HmqeOUFlb5V6uMYVIjyia` |
| `Nebuah/system/memory/` | `1BHERP0HUslcErNfg1EjpHx2cPwRYkdl0` |
| `Nebuah/system/memory/strategies/` | `1gnWPVkO8c4HBPQjEhzVnBTfpsKIF8GJv` |
| `Nebuah/system/memory/strategies/level_1_epics/` | `1qbcLMQzfjaEv69YiCqNx9EtbZzk3qJvt` |
| `Nebuah/system/memory/strategies/level_2_architecture/` | `1YIF2iScp285otUSckU2PhmK_MRKezYQI` |
| `Nebuah/system/memory/strategies/level_3_tactical/` | `19QsdAF3Huq03Z_waeWQKDuPVQNPdfXbn` |
| `Nebuah/system/memory/strategies/level_4_reactive/` | `1AmO5060VgUkoXJABtcjFMZ7MYn2TqHKM` |

## MCP Tool Inventory

All Google Drive operations use MCP tools prefixed with `mcp__claude_ai_Google_Drive__`:

| Tool | Purpose | Sync Role |
|------|---------|-----------|
| `create_file` | Create/upload files (content as base64) | Upload outputs, strategies, project memories |
| `download_file_content` | Download raw binary content | Pull PDFs, binary files |
| `read_file_content` | Read natural language representation | Pull Google Docs, Sheets as text |
| `search_files` | Search with query operators | Cross-project memory queries, file discovery |
| `list_recent_files` | List recently modified files | Detect changes for sync |
| `get_file_metadata` | Get file metadata (title, modified time, MIME type) | Conflict detection, sync state comparison |
| `get_file_permissions` | Get file permissions | Verify access before operations |

## GDrive Registry

### Location

`system/gdrive_registry.json` in the local project root.

### Schema

```json
{
  "version": 1,
  "last_sync": "2026-04-26T10:00:00Z",
  "root_id": "1Ry365y1w5yancVLmosByCnR-MWCpCmxc",
  "mappings": {
    "system/memory/strategies/": {
      "gdrive_id": "1gnWPVkO8c4HBPQjEhzVnBTfpsKIF8GJv",
      "type": "folder",
      "last_synced": "2026-04-26T10:00:00Z"
    },
    "system/memory/strategies/level_1_epics/": {
      "gdrive_id": "1qbcLMQzfjaEv69YiCqNx9EtbZzk3qJvt",
      "type": "folder",
      "last_synced": "2026-04-26T10:00:00Z"
    },
    "system/memory/strategies/_negative_constraints.md": {
      "gdrive_id": "file_id_here",
      "type": "file",
      "mime_type": "text/markdown",
      "last_synced": "2026-04-26T10:00:00Z",
      "local_hash": "sha256:abc123..."
    },
    "projects/CaseName/input/": {
      "gdrive_id": "folder_id_here",
      "type": "folder",
      "last_synced": "2026-04-26T09:30:00Z"
    }
  }
}
```

### Registry Fields

| Field | Type | Description |
|-------|------|-------------|
| `version` | number | Schema version for forward compatibility |
| `last_sync` | string | ISO 8601 timestamp of the last full sync operation |
| `root_id` | string | Google Drive ID of the `Nebuah/` root folder |
| `mappings` | object | Maps local relative paths to GDrive metadata |
| `mappings[path].gdrive_id` | string | Google Drive file or folder ID |
| `mappings[path].type` | string | `"file"` or `"folder"` |
| `mappings[path].mime_type` | string | MIME type (files only) |
| `mappings[path].last_synced` | string | ISO 8601 timestamp of last sync for this entry |
| `mappings[path].local_hash` | string | SHA-256 hash of local file content (files only, for change detection) |

### Registry Operations

**Lookup a GDrive ID by local path**:
```
1. Read("system/gdrive_registry.json")
2. Parse JSON
3. Return mappings[local_path].gdrive_id
```

**Register a new mapping** (after creating a folder/file in GDrive):
```
1. Read("system/gdrive_registry.json")
2. Add entry to mappings
3. Write("system/gdrive_registry.json", updated_json)
```

**Compute local hash** (for change detection):
```
Bash("shasum -a 256 [local_file_path] | cut -d ' ' -f 1")
```

## Auto-Bootstrap Protocol

The system auto-bootstraps Google Drive on the very first invocation of `/nebuah [goal]`. This runs as **Step 0** of the execution workflow and becomes a no-op on subsequent runs.

### Bootstrap Flow

```
Step 0: GDRIVE AUTO-BOOTSTRAP

1. Check: Does `system/gdrive_registry.json` exist with valid `root.id`?
   ├─ YES → Bootstrap already complete. Skip to Step 1. (no-op)
   └─ NO → Continue:

2. Search Google Drive for existing "Nebuah" root folder:
   mcp__claude_ai_Google_Drive__search_files(
     query: "title = 'Nebuah' and mimeType = 'application/vnd.google-apps.folder'"
   )

3. Evaluate:
   ├─ ONE folder found → Use it. Record root ID.
   ├─ MULTIPLE folders → Ask user ONCE: "Which folder should Nebuah use?"
   │                      (This is the ONLY user question for GDrive setup)
   └─ NONE found → Create root folder:
      mcp__claude_ai_Google_Drive__create_file(
        title: "Nebuah",
        mimeType: "application/vnd.google-apps.folder"
      )

4. Create system sub-folders (search before each to avoid duplicates):
   Root/
   ├── projects/
   └── system/
       └── memory/
           └── strategies/
               ├── level_1_epics/
               ├── level_2_architecture/
               ├── level_3_tactical/
               └── level_4_reactive/

5. Write `system/gdrive_registry.json` with all folder ID mappings.

6. Done. All future runs see the registry and skip this step.
```

### Idempotency

- If the registry already exists → instant skip (no GDrive API calls)
- If some sub-folders already exist in GDrive → search finds them, reuse IDs
- If the user deletes the local registry → re-bootstraps from GDrive (discovers existing folders)
- If the user deletes the GDrive folders → creates fresh ones on next run

### User Interaction

The system asks the user **at most ONE question**, and only in this scenario:
- Multiple folders named "Nebuah" exist in the user's Google Drive
- The question: "I found N folders named 'Nebuah'. Which one should I use?"
- After the user answers, the choice is recorded in the registry and never asked again

In all other cases (0 folders or 1 folder), the system proceeds silently.

## Project Lifecycle with GDrive

### Phase 1: Project Initialization

When `/nebuah [goal]` creates a new project, the Init Phase integrates with GDrive automatically.

#### Step 1: Create local project structure

```
Bash("mkdir -p projects/[ProjectName]/{components/agents,output,memory/{short_term,long_term}}")
```

#### Step 2: Create GDrive project folders

```
# Create project root folder in GDrive
mcp__claude_ai_Google_Drive__create_file(
  title: "[ProjectName]",
  mimeType: "application/vnd.google-apps.folder",
  parentId: "1XQCMY-Hv_2pbJKgKWzyj6LMIQ23ylydl"   # Nebuah/projects/
)
# Returns: { id: "new_project_folder_id" }

# Create sub-folders
mcp__claude_ai_Google_Drive__create_file(
  title: "input",
  mimeType: "application/vnd.google-apps.folder",
  parentId: "new_project_folder_id"
)

mcp__claude_ai_Google_Drive__create_file(
  title: "output",
  mimeType: "application/vnd.google-apps.folder",
  parentId: "new_project_folder_id"
)

mcp__claude_ai_Google_Drive__create_file(
  title: "memory",
  mimeType: "application/vnd.google-apps.folder",
  parentId: "new_project_folder_id"
)
# Then create memory/long_term/ inside the memory folder
```

#### Step 3: Register mappings

Update `system/gdrive_registry.json` with all new folder IDs.

#### Step 4: Check for pre-existing input documents

```
# Search for files in the project's input folder
mcp__claude_ai_Google_Drive__search_files(
  query: "parentId = '[input_folder_id]'"
)
```

If files exist, download them using the Input Download Protocol (see below).

### Phase 2: Input Download Protocol

For each file found in the GDrive `input/` folder:

**Step 1: Get metadata to determine MIME type**

```
mcp__claude_ai_Google_Drive__get_file_metadata(
  fileId: "[file_id]"
)
```

**Step 2: Download based on MIME type**

| Source MIME Type | Download Method | Local Format |
|------------------|----------------|--------------|
| `application/vnd.google-apps.document` | `read_file_content` | `.md` (natural language text) |
| `application/vnd.google-apps.spreadsheet` | `read_file_content` | `.md` (tabular text) |
| `application/pdf` | `download_file_content` | `.pdf` (raw binary) |
| `text/plain` | `download_file_content` | `.txt` (raw text) |
| `text/markdown` | `download_file_content` | `.md` (raw markdown) |
| `application/vnd.openxmlformats-officedocument.wordprocessingml.document` | `read_file_content` | `.md` (extracted text) |
| `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` | `read_file_content` | `.md` (extracted tables) |

**Google Docs download pattern**:
```
# Read Google Doc as natural language text
mcp__claude_ai_Google_Drive__read_file_content(
  fileId: "[google_doc_id]"
)
# Returns: natural language representation of the document

# Write to local filesystem
Write("projects/[ProjectName]/input/[doc_title].md", content)
```

**PDF / binary download pattern**:
```
# Download raw content
mcp__claude_ai_Google_Drive__download_file_content(
  fileId: "[pdf_file_id]"
)
# Returns: raw binary data

# Write to local filesystem using Bash (for binary content)
# The content from download_file_content is base64-encoded
Bash("echo '[base64_content]' | base64 -d > projects/[ProjectName]/input/[filename].pdf")
```

**Google Sheets download pattern**:
```
# Read spreadsheet as structured text
mcp__claude_ai_Google_Drive__read_file_content(
  fileId: "[spreadsheet_id]"
)
# Returns: tabular representation

# Write to local filesystem
Write("projects/[ProjectName]/input/[sheet_title].md", content)
```

### Phase 3: Local Work Phase

All work happens locally. No GDrive calls during active execution.

- Agents read from `projects/[ProjectName]/input/`
- Agents write to `projects/[ProjectName]/output/`
- Traces are written to `system/memory/traces/`
- Dream Engine writes strategies to `system/memory/strategies/`

### Phase 4: Output Upload Protocol

After task completion (before or after dream consolidation), upload outputs to GDrive.

**For each file in `projects/[ProjectName]/output/`**:

**Step 1: Read local file and encode**:
```
# Read the file content
Read("projects/[ProjectName]/output/[filename].md")

# Base64 encode for upload
Bash("base64 -i projects/[ProjectName]/output/[filename].md")
```

**Step 2: Upload to GDrive**:
```
mcp__claude_ai_Google_Drive__create_file(
  title: "[filename].md",
  mimeType: "text/markdown",
  parentId: "[project_output_folder_id]",
  content: "[base64_encoded_content]"
)
```

**Step 3: Register the file mapping**:
```
# Update registry with new file ID and local hash
Read("system/gdrive_registry.json")
# Add: "projects/[ProjectName]/output/[filename].md": { gdrive_id, type: "file", ... }
Write("system/gdrive_registry.json", updated_json)
```

**Creating a Google Doc from Markdown output**:
```
mcp__claude_ai_Google_Drive__create_file(
  title: "[Document Title]",
  mimeType: "text/plain",
  parentId: "[project_output_folder_id]",
  content: "[base64_encoded_markdown_content]"
)
# text/plain is automatically converted to Google Docs format
# To prevent conversion, set disableConversionToGoogleType: true
```

## Memory Sync Protocol

### Upload: Local Strategies to GDrive

Triggered after dream consolidation or via `gdrive push`.

**Step 1: Identify changed strategies**:
```
# List all local strategy files
Glob(pattern: "system/memory/strategies/level_*/**/*.md")

# For each file, compare local hash with registry hash
Bash("shasum -a 256 [strategy_file] | cut -d ' ' -f 1")
# Compare with mappings[path].local_hash in registry
```

**Step 2: Upload new or changed strategies**:
```
# Read and encode the strategy file
Bash("base64 -i system/memory/strategies/level_2_architecture/[strategy].md")

# Upload to the matching GDrive folder
mcp__claude_ai_Google_Drive__create_file(
  title: "[strategy].md",
  mimeType: "text/markdown",
  parentId: "[level_2_architecture_folder_id]",    # From registry
  content: "[base64_encoded_content]"
)
```

**Step 3: Upload constraints and dream journal**:
```
# Upload _negative_constraints.md
Bash("base64 -i system/memory/strategies/_negative_constraints.md")
mcp__claude_ai_Google_Drive__create_file(
  title: "_negative_constraints.md",
  mimeType: "text/markdown",
  parentId: "1gnWPVkO8c4HBPQjEhzVnBTfpsKIF8GJv",  # strategies/ folder
  content: "[base64_encoded_content]"
)

# Upload _dream_journal.md
Bash("base64 -i system/memory/strategies/_dream_journal.md")
mcp__claude_ai_Google_Drive__create_file(
  title: "_dream_journal.md",
  mimeType: "text/markdown",
  parentId: "1gnWPVkO8c4HBPQjEhzVnBTfpsKIF8GJv",
  content: "[base64_encoded_content]"
)
```

**Step 4: Update registry with new hashes and sync timestamps**.

### Download: GDrive Strategies to Local

Triggered before a task (optional) or via `gdrive pull`.

**Step 1: List strategies in each GDrive level folder**:
```
# For each level folder
mcp__claude_ai_Google_Drive__search_files(
  query: "parentId = '1qbcLMQzfjaEv69YiCqNx9EtbZzk3qJvt'"
)
# Repeat for level_2, level_3, level_4 folder IDs
```

**Step 2: Compare with local files**:
```
# For each GDrive file, check if local version exists
mcp__claude_ai_Google_Drive__get_file_metadata(
  fileId: "[strategy_file_id]"
)
# Compare modifiedTime with registry last_synced
```

**Step 3: Download newer files**:
```
mcp__claude_ai_Google_Drive__read_file_content(
  fileId: "[strategy_file_id]"
)
# Write to local path
Write("system/memory/strategies/level_[N]_[name]/[strategy].md", content)
```

**Step 4: Download constraints and dream journal**:
```
# Search for constraints file
mcp__claude_ai_Google_Drive__search_files(
  query: "title = '_negative_constraints.md' and parentId = '1gnWPVkO8c4HBPQjEhzVnBTfpsKIF8GJv'"
)
# Download and write locally
mcp__claude_ai_Google_Drive__read_file_content(fileId: "[constraints_file_id]")
Write("system/memory/strategies/_negative_constraints.md", content)

# Repeat for _dream_journal.md
```

### Project Memory Upload

After dream consolidation writes project-specific learnings to `projects/[ProjectName]/memory/long_term/`:

```
# List local project memory files
Glob(pattern: "projects/[ProjectName]/memory/long_term/*.md")

# For each file, encode and upload
Bash("base64 -i projects/[ProjectName]/memory/long_term/[file].md")
mcp__claude_ai_Google_Drive__create_file(
  title: "[file].md",
  mimeType: "text/markdown",
  parentId: "[project_long_term_folder_id]",
  content: "[base64_encoded_content]"
)
```

## Cross-Project Memory Access

Sub-agents can query Google Drive for strategies and learnings from other projects. This is the primary mechanism for knowledge transfer between engagements.

### Pattern 1: Keyword-Based Strategy Search

When an agent needs strategies beyond the local filesystem (e.g., working on a new machine or a new project that may benefit from past work):

```
# Search across all strategies for relevant keywords
mcp__claude_ai_Google_Drive__search_files(
  query: "fullText contains 'indemnification' and parentId = '1gnWPVkO8c4HBPQjEhzVnBTfpsKIF8GJv'"
)
# Returns: list of matching strategy files

# Read each match
mcp__claude_ai_Google_Drive__read_file_content(
  fileId: "[matching_strategy_id]"
)
# Parse YAML frontmatter, score, and inject into agent context
```

### Pattern 2: Level-Scoped Strategy Search

When searching for strategies at a specific hierarchy level:

```
# Search only in level_2_architecture
mcp__claude_ai_Google_Drive__search_files(
  query: "fullText contains 'regulatory compliance' and parentId = '1YIF2iScp285otUSckU2PhmK_MRKezYQI'"
)
```

### Pattern 3: Cross-Project Learning Search

When seeking learnings from a specific past project:

```
# First, find the project folder
mcp__claude_ai_Google_Drive__search_files(
  query: "title = '[PastProjectName]' and parentId = '1XQCMY-Hv_2pbJKgKWzyj6LMIQ23ylydl'"
)
# Returns: project folder ID

# Then search its long_term memory
mcp__claude_ai_Google_Drive__search_files(
  query: "parentId = '[past_project_long_term_folder_id]'"
)

# Read relevant memories
mcp__claude_ai_Google_Drive__read_file_content(
  fileId: "[memory_file_id]"
)
```

### Pattern 4: Recent Global Changes

When checking what has changed across the entire Nebuah Drive since last sync:

```
mcp__claude_ai_Google_Drive__list_recent_files(
  orderBy: "lastModified",
  pageSize: 20
)
# Filter results to files within the Nebuah root
```

### Integration with QueryMemory

The `QueryMemory` function (see `QueryMemoryTool.md`) is extended with a GDrive fallback:

```
QueryMemory (extended):
1. Search local strategies (standard flow)
2. If fewer than 3 matches found locally AND gdrive_registry.json exists:
   a. Search GDrive strategies using cross-project patterns above
   b. Download matching strategies to local cache
   c. Merge with local results, re-score, return top 5
```

## Sync Commands

### `gdrive pull [project]`

Downloads input documents from GDrive to local project directory.

**Workflow**:
```
1. Read("system/gdrive_registry.json") → get project's input folder ID
2. mcp__claude_ai_Google_Drive__search_files(query: "parentId = '[input_folder_id]'")
3. For each file:
   a. get_file_metadata → determine MIME type
   b. download_file_content or read_file_content → get content
   c. Write to projects/[project]/input/[filename]
4. Update registry with sync timestamps
5. Report: "[N] files downloaded from GDrive to projects/[project]/input/"
```

**Without project argument** (`gdrive pull`):
```
Pull system memory (strategies, constraints, dream journal) from GDrive to local.
```

### `gdrive push [project]`

Uploads output files and project memories to GDrive.

**Workflow**:
```
1. Read("system/gdrive_registry.json") → get project's output and memory folder IDs
2. Glob("projects/[project]/output/*") → list local outputs
3. For each output file:
   a. Compute local hash
   b. Compare with registry hash (skip if unchanged)
   c. base64 encode and upload via create_file
4. Glob("projects/[project]/memory/long_term/*") → list local memories
5. For each memory file:
   a. Same hash-compare-upload cycle
6. Update registry
7. Report: "[N] files uploaded to GDrive projects/[project]/"
```

**Without project argument** (`gdrive push`):
```
Push system memory (strategies, constraints, dream journal) to GDrive.
```

### `gdrive sync`

Full bidirectional synchronization of system memory.

**Workflow**:
```
1. Read("system/gdrive_registry.json")
2. PULL phase:
   a. List all files in GDrive system/memory/strategies/ (all levels)
   b. For each GDrive file:
      - Compare modifiedTime with registry last_synced
      - If GDrive is newer: download and overwrite local
3. PUSH phase:
   a. List all local strategy files
   b. For each local file:
      - Compute hash, compare with registry
      - If local has changed since last sync: upload to GDrive
4. MERGE phase (constraints only):
   a. Download GDrive _negative_constraints.md
   b. Read local _negative_constraints.md
   c. Merge: append constraints from GDrive that don't exist locally
   d. Upload merged version back to GDrive
   e. Write merged version locally
5. Update registry with all new timestamps and hashes
6. Report sync summary
```

### `gdrive status`

Display current synchronization state.

**Workflow**:
```
1. Read("system/gdrive_registry.json")
2. For each mapping:
   a. Compute current local hash
   b. Compare with registry hash
   c. Mark as: synced | local_changed | not_tracked
3. Report:
   - Last full sync: [timestamp]
   - System memory: [N] strategies, [synced/changed/untracked] status
   - Projects: [list with per-project sync state]
   - Pending uploads: [count]
   - Pending downloads: [unknown until GDrive is queried]
4. Optionally query GDrive for remote changes:
   mcp__claude_ai_Google_Drive__list_recent_files(orderBy: "lastModified", pageSize: 10)
   - Compare with registry to identify remote changes
```

## Supported File Formats

### Input Formats (GDrive to Local)

| Format | GDrive MIME Type | Local Storage | Download Tool |
|--------|------------------|---------------|---------------|
| Google Docs | `application/vnd.google-apps.document` | `.md` | `read_file_content` |
| Google Sheets | `application/vnd.google-apps.spreadsheet` | `.md` | `read_file_content` |
| PDF | `application/pdf` | `.pdf` | `download_file_content` |
| Plain Text | `text/plain` | `.txt` | `download_file_content` |
| Markdown | `text/markdown` | `.md` | `download_file_content` |
| Word (.docx) | `application/vnd.openxmlformats-officedocument.wordprocessingml.document` | `.md` | `read_file_content` |
| Excel (.xlsx) | `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` | `.md` | `read_file_content` |

### Output Formats (Local to GDrive)

| Format | Upload MIME Type | GDrive Result | Notes |
|--------|------------------|---------------|-------|
| Markdown | `text/plain` | Google Doc | Auto-converted unless `disableConversionToGoogleType: true` |
| Raw Markdown | `text/markdown` + `disableConversionToGoogleType: true` | `.md` file | Preserves YAML frontmatter |
| CSV | `text/csv` | Google Sheet | Auto-converted |
| PDF | `application/pdf` | PDF file | No conversion |

### Strategy Files (Bidirectional)

Strategies MUST be uploaded with `disableConversionToGoogleType: true` to preserve YAML frontmatter:

```
mcp__claude_ai_Google_Drive__create_file(
  title: "strat_2_regulatory-framework.md",
  mimeType: "text/plain",
  parentId: "[level_2_folder_id]",
  content: "[base64_encoded_strategy_with_yaml_frontmatter]",
  disableConversionToGoogleType: true
)
```

This is critical. If a strategy file is converted to Google Docs format, the YAML frontmatter will be corrupted and the strategy becomes unparseable by `QueryMemory`.

## Conflict Resolution

### Resolution Rules

| Resource Type | Conflict Rule | Rationale |
|---------------|---------------|-----------|
| Strategies | GDrive wins | GDrive is the shared source of truth across machines |
| Project outputs | Local wins | Most recent work is always local |
| Negative constraints | Merge (union) | Constraints are append-only; no constraint should be lost |
| Dream journal | GDrive wins | Journal is append-only; GDrive has the most complete history |
| Seed strategies | Never overwritten | `_seeds/` are immutable on both local and GDrive |
| Project inputs | GDrive wins | Inputs originate from GDrive (user uploads) |
| Registry | Local wins | Registry reflects the local machine's sync state |

### Conflict Detection

A conflict exists when:
```
local_hash != registry_hash AND gdrive_modified_time > registry_last_synced
```

This means both sides have changed since the last sync.

### Merge Protocol for Constraints

When `_negative_constraints.md` has conflicts:

```
1. Download GDrive version:
   mcp__claude_ai_Google_Drive__read_file_content(fileId: "[constraints_id]")

2. Read local version:
   Read("system/memory/strategies/_negative_constraints.md")

3. Parse both versions into individual constraint entries
   (Each constraint is a markdown section starting with ### or a list item with severity)

4. Create union set:
   - Keep all local constraints
   - Append GDrive constraints whose text does not appear in local
   - Preserve severity levels from the source that has the higher severity

5. Write merged result locally:
   Write("system/memory/strategies/_negative_constraints.md", merged)

6. Upload merged result to GDrive:
   Bash("base64 -i system/memory/strategies/_negative_constraints.md")
   mcp__claude_ai_Google_Drive__create_file(
     title: "_negative_constraints.md",
     mimeType: "text/plain",
     parentId: "1gnWPVkO8c4HBPQjEhzVnBTfpsKIF8GJv",
     content: "[base64_merged]",
     disableConversionToGoogleType: true
   )

7. Update registry with new hash and timestamp
```

## Error Handling

### Common Failures

| Error | Cause | Recovery |
|-------|-------|----------|
| File not found (404) | GDrive file deleted or ID stale | Remove from registry, re-search by title |
| Permission denied (403) | Insufficient access | Log warning, skip file, notify user |
| Quota exceeded (429) | Too many API calls | Back off, retry after delay, batch remaining |
| Upload size limit | File exceeds GDrive limits | Split or compress, warn user |
| Network failure | Connectivity issue | Log as PARTIAL sync, retry on next command |

### Recovery Protocol

When a GDrive operation fails:

```
1. Log the failure in the current trace:
   [MCP: mcp__claude_ai_Google_Drive__create_file] -> FAILED: [error message]

2. If the operation is part of a batch:
   - Continue with remaining operations
   - Report partial sync at the end

3. If the registry is left in an inconsistent state:
   - Mark the affected entry with "sync_error": "[error message]"
   - Next sync command will detect and retry

4. NEVER silently drop files — always report what failed and why
```

## Integration with Nebuah Lifecycle

### Where GDrive Fits in the Execution Pipeline

```
/nebuah [goal]
│
├─ Step 0: GDRIVE AUTO-BOOTSTRAP (first run only — no-op after)
│  └─ Search/create Nebuah root folder → create system sub-folders → write registry
│
├─ Step 1: MEMORY QUERY
│  └─ Cross-project memory from GDrive (automatic)
│
├─ Step 2: ANALYZE & PLAN
│
├─ Step 3: CREATE PROJECT STRUCTURE + GDRIVE MIRROR
│  └─ Create local folders + GDrive project folders + download inputs (all automatic)
│
├─ Step 4: CREATE SPECIALIZED AGENTS
│
├─ Step 5: EXECUTE (local only, no GDrive calls)
│
├─ Step 6: PRODUCE OUTPUT + UPLOAD TO GDRIVE
│  └─ Save locally → auto-upload to GDrive output/ folder
│
├─ Step 7: CONSOLIDATE + SYNC LEARNINGS
│  └─ Dream Engine → auto-upload strategies/constraints/journal to GDrive
│
└─ Step 8: REPORT
```

### Automatic Sync Triggers (All Seamless)

| Event | Sync Action | Direction | User Action |
|-------|-------------|-----------|-------------|
| First `/nebuah` invocation | Auto-bootstrap root folder + system structure | Setup | None |
| New project created | Create GDrive folders, download inputs | Pull | None |
| Output produced | Upload outputs to GDrive | Push | None |
| Dream consolidation complete | Upload new/updated strategies | Push | None |
| `gdrive pull` command (manual) | Download strategies + project inputs | Pull | Manual override |
| `gdrive push` command (manual) | Upload outputs + memories | Push | Manual override |
| `gdrive sync` command (manual) | Bidirectional strategy sync | Both | Manual override |

### Trace Logging for GDrive Operations

GDrive operations should be logged in traces at L4 level:

```markdown
### Time: 2026-04-26T14:30:00Z
**Trace ID:** tr_1745678200000_d4f1
**Level:** 4
**Parent:** tr_1745678000000_a1b2
**Goal:** Upload project outputs to Google Drive
**Strategy:** null
**Outcome:** SUCCESS
**Reason:** 3 files uploaded to GDrive projects/MandA-DueDiligence/output/
**Duration:** 8500
**Confidence:** 1.0

**Actions:**
1. [Read: system/gdrive_registry.json] -> Loaded registry with 12 mappings
2. [Glob: projects/MandA-DueDiligence/output/*] -> Found 3 output files
3. [Bash: base64 encode due-diligence-report.md] -> Encoded 24KB
4. [MCP: create_file due-diligence-report.md] -> Uploaded, ID: abc123
5. [Bash: base64 encode risk-assessment.md] -> Encoded 18KB
6. [MCP: create_file risk-assessment.md] -> Uploaded, ID: def456
7. [Bash: base64 encode executive-summary.md] -> Encoded 6KB
8. [MCP: create_file executive-summary.md] -> Uploaded, ID: ghi789
9. [Write: system/gdrive_registry.json] -> Updated 3 file mappings
---
```

## Security and Access

### Permissions Model

- The Nebuah GDrive root folder should be shared with all team members who need access
- Project folders inherit permissions from `Nebuah/projects/`
- System memory inherits from `Nebuah/system/`
- Individual file permissions are not managed by Nebuah — use GDrive's native sharing

### Sensitive Data Handling

1. **NEVER upload trace files** to GDrive. Traces contain raw execution details and may include sensitive client data. Only consolidated strategies (which are abstracted) are synced.
2. **NEVER upload agent definition files** to GDrive. Agent prompts may contain case-specific context.
3. **Project inputs stay in GDrive** — downloaded locally for processing but the GDrive copy is the canonical source.
4. **Strategy files are safe to sync** — they contain abstracted patterns, not case-specific data.

### What Gets Synced (Allowlist)

| Local Path | Synced? | Direction |
|------------|---------|-----------|
| `system/memory/strategies/level_*/` | Yes | Bidirectional |
| `system/memory/strategies/_negative_constraints.md` | Yes | Bidirectional (merge) |
| `system/memory/strategies/_dream_journal.md` | Yes | Bidirectional |
| `system/memory/strategies/_seeds/` | No | Never (immutable local) |
| `system/memory/traces/` | No | Never (sensitive) |
| `projects/[name]/input/` | Yes | Pull only |
| `projects/[name]/output/` | Yes | Push only |
| `projects/[name]/memory/long_term/` | Yes | Push only |
| `projects/[name]/memory/short_term/` | No | Never (sensitive) |
| `projects/[name]/components/agents/` | No | Never (sensitive) |
| `system/gdrive_registry.json` | No | Local only |

## Initialization Bootstrap

Bootstrap is now **fully automatic** via Step 0 of the execution workflow (see Auto-Bootstrap Protocol above).

**When `system/gdrive_registry.json` does not exist**, the system:
1. Searches GDrive for existing "Nebuah" folder (search before create)
2. Creates root + system sub-folders if needed
3. Writes the registry with all discovered/created folder IDs
4. Proceeds with the normal execution flow

**The user is never asked to run any setup commands.** The only possible user interaction is disambiguation if multiple "Nebuah" folders exist.

For machines where the registry already exists (e.g., from a previous session), Step 0 is a no-op (just reads the file and confirms `root.id` is present).

## Tool Limitations and Workarounds

| Limitation | Workaround |
|-----------|------------|
| `create_file` content must be base64 | Always encode via `Bash("base64 -i [file]")` before upload |
| `read_file_content` may truncate large files | For large files, use `download_file_content` with `exportMimeType` |
| `search_files` returns paginated results | Use `pageToken` to iterate through all results |
| No "update file" MCP tool — only create | Upload creates a new file; manually manage duplicates by title search first |
| Google Docs conversion strips YAML frontmatter | Always use `disableConversionToGoogleType: true` for strategy files |
| `read_file_content` format is not stable | Do not hard-code parsing of its output; treat as best-effort text |
| Binary files (PDF) cannot be written via `Write` tool | Use `Bash` to decode base64 and write binary content |

### Duplicate Prevention

Since the MCP tools do not support "update in place", prevent duplicates before uploading:

```
# Before creating a file, search for existing file with same title in same folder
mcp__claude_ai_Google_Drive__search_files(
  query: "title = '[filename]' and parentId = '[folder_id]'"
)

# If found: the file already exists
#   Option A: Skip upload (if content unchanged based on hash)
#   Option B: Note the old file ID, upload new version, update registry
#             (Old version remains in GDrive but registry points to new one)

# If not found: proceed with create_file
```
