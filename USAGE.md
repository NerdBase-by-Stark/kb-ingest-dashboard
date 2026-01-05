# Usage Guide

## Table of Contents

- [Basic Usage](#basic-usage)
- [Dashboard Panels](#dashboard-panels)
- [Session Awareness](#session-awareness)
- [Workflows](#workflows)
- [Command-Line Options](#command-line-options)
- [Interpreting the Display](#interpreting-the-display)
- [Tips and Best Practices](#tips-and-best-practices)
- [Common Scenarios](#common-scenarios)

## Basic Usage

### Starting the Dashboard

```bash
# Default view - all panels including logs
kb-ingest-dashboard

# Clean view - hide log panel
kb-ingest-dashboard --hide-logs
kb-ingest-dashboard --no-logs  # Alias
```

### Stopping the Dashboard

Press **Ctrl+C** at any time to quit.

## Dashboard Panels

The dashboard consists of 7 panels arranged across the screen:

### Layout Overview

```
╭─────────────────────────────────────────────────────────────────────╮
│                         HEADER (timestamp)                            │
├──────────────────────────┬───────────────────────────────────────────┤
│                          │           RIGHT COLUMN (40%)              │
│      LEFT COLUMN (60%)    ├───────────────────────────────────────────┤
│                          │                                           │
│  ┌────────────────────┐   │  ┌───────────────────────────────────┐  │
│  │   Progress Panel   │   │  │       Ingestion Status           │  │
│  │     (size=6)        │   │  │          (size=15)               │  │
│  └────────────────────┘   │  └───────────────────────────────────┘  │
│                          │  ┌───────────────────────────────────┐  │
│  ┌────────────────────┐   │  │         LLM Usage                │  │
│  │     Log Panel      │   │  │          (size=10)               │  │
│  │     (ratio=1)      │   │  └───────────────────────────────────┘  │
│  └────────────────────┘   │  ┌───────────────────────────────────┐  │
│                          │  │       System Resources             │  │
│                          │  │          (size=5)                │  │
│                          │  └───────────────────────────────────┘  │
│                          │  ┌───────────────────────────────────┐  │
│                          │  │         Info Panel                │  │
│                          │  │          (size=5)                │  │
│                          │  └───────────────────────────────────┘  │
└──────────────────────────┴───────────────────────────────────────────┘
```

### 1. Progress Panel (Left, Top)

Shows overall ingestion progress:

```
╭──────────────── Good Progress ────────────────────────────────╮
│  690/3,427 chunks  (20.1%)                                          │
│  ███████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░   │
╰─────────────────────────────────────────────────────────────────╯
```

| Element | Description |
|---------|-------------|
| **Current/Total Chunks** | Documents processed vs. total document count |
| **Percentage** | Completion percentage |
| **Progress Bar** | Visual representation (fills left to right) |
| **Label** | Status indicator: Starting, In Progress, Good Progress, Almost Done |

### 2. Ingestion Status Panel (Right, Top)

Detailed metrics about the current session:

```
╭─────────────── Ingestion Status ───────────────────────────────╮
│  Status:                        RUNNING                             │
│  Progress:            690/3,427 (20.1%)                          │
│  Elapsed:                       30h 45m                            │
│  ETA:                          125h 30m                            │
│  Rate:                     21 chunks/hr                            │
│  Entities:            3,633 (5.4/chunk)                           │
│  Relationships:       2,515 (3.8/chunk)                           │
│  Est. LLM Calls/Model:                  2,760                       │
│  Est. Tokens/Model:                 2,070,000                       │
╰─────────────────────────────────────────────────────────────────╯
```

| Field | Description |
|-------|-------------|
| **Status** | `RUNNING` (green) if ingestion process detected, `STOPPED` (red) otherwise |
| **Progress** | Chunks processed with percentage |
| **Elapsed** | Time since current session started |
| **ETA** | Estimated time until completion |
| **Rate** | Processing speed (chunks per hour) |
| **Entities** | Total entities extracted (avg per chunk) |
| **Relationships** | Total relationships found (avg per chunk) |
| **Est. LLM Calls/Model** | Estimated API calls per model |
| **Est. Tokens/Model** | Estimated token usage per model |

### 3. LLM Usage Panel (Right, Middle)

Per-model tracking of LLM usage:

```
╭───────────── LLM Usage (est. 750 tok/call) ─────────────────────╮
│  Model              Entities  Est. LLM Calls  Est. Tokens        │
│  ──────────────────────────────────────────────────────────────  │
│  qwen2.5:7b             3,234           690         517,500        │
│  mistral:latest        2,718           690         517,500        │
│  llama3.1:8b           2,654           690         517,500        │
│  qwen3                     0           690         517,500        │
╰─────────────────────────────────────────────────────────────────╯
```

| Column | Description |
|--------|-------------|
| **Model** | LLM model name (as recorded in checkpoint) |
| **Entities** | Number of entities extracted by this model |
| **Est. LLM Calls** | Estimated API calls (chunks processed × 1) |
| **Est. Tokens** | Estimated token usage (~750 tokens/call) |

**Notes**:
- `qwen3` is used for relationship extraction (often shows 0 entities)
- Ensemble models (e.g., `llama3.1:8b,qwen2.5:7b`) are split per-model
- Token estimate assumes ~750 tokens per call based on ~3000 char chunks

### 4. System Resources Panel (Right, Bottom)

Real-time system resource monitoring:

```
╭───────────────── System Resources ─────────────────────────────╮
│       CPU:    ░░░░░░░░░░ 6%                                       │
│    Memory:    ██░░░░░░░░ 30.0GB (25%)                             │
│     Disk:    ░░░░░░░░░░ 273GB used                                │
╰─────────────────────────────────────────────────────────────────╯
```

| Resource | Description |
|----------|-------------|
| **CPU** | Current CPU usage percentage with visual bar |
| **Memory** | RAM used (total GB and percentage) |
| **Disk** | Disk space used by output directory |

### 5. Log Panel (Left, Bottom)

Live ingestion logs with color coding:

```
╭────────────────────────────────── Logs ────────────────────────╮
│  Chunk 688/3427: _Content_Application_...                       │
│  [12:04:49] Extracted 17 validated entities                     │
│  [12:05:23] Found 5 relationships                                │
│  Chunk 689/3427: _Content_Control_...                           │
│  [12:06:12] Extracted 8 validated entities                      │
╰─────────────────────────────────────────────────────────────────╯
```

| Color | Type of Message |
|-------|----------------|
| **Cyan** | Chunk progress ("Chunk 688/3427: ...") |
| **Green** | Entity extraction ("Extracted N validated entities") |
| **Magenta** | Relationship findings ("Found N relationships") |
| **Dim** | General information |

### 6. Info Panel (Right, Bottom)

Session information and controls:

```
╭───────────────────────── Info ─────────────────────────────────╮
│  Output: /home/user/ai/knowledge-base/qsys-full-extract         │
│  Log: pipeline.log                                               │
│                                                                   │
│  Press Ctrl+C to quit                                             │
╰─────────────────────────────────────────────────────────────────╯
```

## Session Awareness

The dashboard is **session-aware** - it only tracks data from the current ingestion session.

### How Sessions Are Detected

Sessions are identified by scanning the log file for markers:

1. Lines with 5+ consecutive equals: `====`
2. Lines containing "PIPELINE" and "v3"

Example session start:
```
[2026-01-04 22:03:53] ============================================================
[2026-01-04 22:03:53] QUALITY KB INGESTION PIPELINE v3
[2026-01-04 22:03:53] ============================================================
```

The dashboard uses the **most recent** marker as the current session start.

### Why Session Awareness Matters

**Without session filtering:**
- Entities from all previous runs would be counted
- Progress would show cumulative total (confusing)
- ETA would be calculated from first-ever run (inaccurate)

**With session filtering:**
- Only entities from current session are counted
- Progress reflects the current run only
- Checkpoints from before session start are ignored
- Accurate timing for current session

### Visual Example

```
First run (Morning):
  [08:00] PIPELINE v3 start
  [08:00-12:00] Processed 1000 chunks, extracted 5000 entities

Dashboard restart at 14:00:
  Session detected: 08:00 start (first marker)
  Shows: 1000 chunks, 5000 entities ✓ CORRECT

Second run (Afternoon):
  [14:00] PIPELINE v3 start
  [14:00-16:00] Processed 500 chunks, extracted 2500 entities

Dashboard restart at 16:00:
  Session detected: 14:00 start (latest marker)
  Shows: 500 chunks, 2500 entities ✓ CORRECT (not 7500!)
```

## Workflows

### Monitoring a New Ingestion

**Terminal 1: Start ingestion**
```bash
cd ~/ai/knowledge-base
python scripts/kb_ingest_v3.py --source ~/docs/qsys --output qsys-full-extract
```

**Terminal 2: Start dashboard**
```bash
kb-ingest-dashboard
```

### Monitoring with Hidden Logs

For a cleaner view on smaller screens:
```bash
kb-ingest-dashboard --hide-logs
```

### Resuming After Dashboard Closure

If you close the dashboard, simply restart it:
```bash
kb-ingest-dashboard
```

It will automatically:
- Detect the running ingestion process
- Find the latest checkpoint
- Resume from session start
- Show current progress

### Post-Ingestion Review

After ingestion completes:
```bash
kb-ingest-dashboard --hide-logs
```

Useful for reviewing final statistics without log clutter.

## Command-Line Options

| Option | Description |
|--------|-------------|
| (none) | Show all panels including logs |
| `--hide-logs` | Hide log panel for cleaner view |
| `--no-logs` | Alias for `--hide-logs` |
| `--help` | Show help message |
| `--version` | Show version info |

## Interpreting the Display

### Status Indicators

| Status | Meaning | Color |
|--------|---------|-------|
| `RUNNING` | Ingestion process active (kb_ingest_v3.py detected) | Green |
| `STOPPED` | No process detected | Red |

### Progress Labels

| Label | Range | Color |
|-------|-------|-------|
| `Starting` | 0-25% | Red |
| `In Progress` | 25-50% | Yellow |
| `Good Progress` | 50-75% | Green |
| `Almost Done` | 75-100% | Bright Green |

### ETA Calculation

```
rate = elapsed_seconds / current_chunks
eta_seconds = (total_chunks - current_chunks) × rate
```

The ETA becomes more accurate as ingestion progresses (minimum ~10% completion recommended).

### Understanding LLM Usage

The LLM Usage panel shows **entity counts per model**, not total API calls.

| Model | Entities | Meaning |
|-------|----------|---------|
| `llama3.1:8b` | 2,654 | This model extracted 2,654 entities |
| `qwen2.5:7b` | 3,234 | This model extracted 3,234 entities |
| `qwen3` | 0 | Relationship extraction (entities counted separately) |

**Ensemble models**: If a checkpoint shows `"llama3.1:8b,qwen2.5:7b"`, both models get credit for that entity.

## Tips and Best Practices

1. **Use two terminals**: One for ingestion, one for monitoring
2. **Hide logs for cleaner view**: Use `--hide-logs` on smaller screens
3. **Monitor LLM costs**: Watch token estimates to manage API spending
4. **Check disk space**: Ensure adequate space before large ingestions
5. **Session restart**: If ingestion restarts, the dashboard automatically tracks the new session
6. **Trust checkpoints**: If dashboard shows 0 entities, ensure checkpoints are being written
7. **Check Status**: Green `RUNNING` = pipeline active; Red `STOPPED` = pipeline stopped

## Common Scenarios

### Large Dataset Ingestion

For datasets with 10,000+ chunks:
- ETA becomes more accurate after 10% completion
- Monitor "Rate" (chunks/hr) to estimate total time
- Watch memory usage if running locally

### Multi-Model Extraction

When using multiple LLMs:
- LLM Usage panel shows per-model entity counts
- Ensemble models are split for accurate tracking
- Token estimates are per-model, not total

### Interrupted and Resumed Ingestion

The dashboard handles resumption correctly:
- Session start detected from log markers
- Progress counts from session start (not cumulative)
- Checkpoints before session are ignored

### Checking Completion

When ingestion finishes:
1. Status changes to `STOPPED` (red)
2. Progress shows `X/Total chunks (100.0%)`
3. ETA shows "0s" or similar
4. You can close the dashboard safely

---

For more details, see:
- [Installation Guide](INSTALL.md)
- [Architecture](ARCHITECTURE.md)
