# Architecture

## Table of Contents

- [Overview](#overview)
- [System Architecture](#system-architecture)
- [Class Structure](#class-structure)
- [Data Flow](#data-flow)
- [Checkpoint Format](#checkpoint-format)
- [Session Awareness](#session-awareness)
- [Dependencies](#dependencies)
- [Extensibility](#extensibility)

## Overview

The KB Ingest Dashboard is a Python TUI application built with [Rich](https://rich.readthedocs.io/). It monitors knowledge base ingestion by parsing checkpoint files and logs in real-time.

**Key Design Principles:**
- **Session-Aware**: Only tracks current session data
- **Read-Only**: Doesn't interfere with ingestion pipeline
- **Stateless**: Can be started/stopped anytime
- **Auto-Install**: Dependencies install on first run

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    KB Ingest Dashboard v1.0                          │
├─────────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                   │
│  │  Progress   │  │   Status    │  │ LLM Usage   │                   │
│  │   Panel     │  │   Panel     │  │   Panel     │                   │
│  │  (Rich Live) │  │  (Rich Live) │  │  (Rich Live) │                   │
│  └─────────────┘  └─────────────┘  └─────────────┘                   │
│  ┌─────────────┐  ┌─────────────┐                                   │
│  │    Logs    │  │   System    │                                   │
│  │   Panel     │  │   Panel     │                                   │
│  └─────────────┘  └─────────────┘                                   │
├─────────────────────────────────────────────────────────────────────┤
│                   KBIngestDashboard Class                          │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │ Session Detection │ Checkpoint Parser │ Log Reader │ Proc    │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                              │                                      │
│                              ▼                                      │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                    Update Loop (2 Hz)                           │  │
│  │  _find_process() → _get_latest_checkpoint() → _read_logs()   │  │
│  └───────────────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────────┤
│                      Data Sources (Read-Only)                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │ pipeline.log │  │checkpoint_*.json │  /proc/*     │          │
│  │ (logs)       │  │(progress)    │  (system)     │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────────────────────────────────────────────────────┘
```

## Class Structure

### KBIngestDashboard

Main class orchestrating all dashboard functionality.

```python
class KBIngestDashboard:
    """Real-time dashboard for KB ingestion monitoring"""

    def __init__(self, show_logs: bool = True)
    def run(self)

    # Private methods
    def _find_latest_log(self)
    def _get_session_start_from_log(self)
    @property
    def session_start(self)
    def _find_process(self)
    def _parse_checkpoint(self, checkpoint_path)
    def _get_latest_checkpoint(self)
    def _read_logs(self)
    def _update_stats(self)
    def _build_layout(self)
    def _build_progress_panel(self)
    def _build_status_panel(self)
    def _build_model_panel(self)
    def _build_system_panel(self)
    def _build_log_panel(self)
    def _build_info_panel(self)
    def _make_mini_bar(self, pct, color, width)
```

### Key Methods

| Method | Purpose |
|--------|---------|
| `_find_latest_log()` | Locate `pipeline.log` or latest `run_*.log` |
| `_get_session_start_from_log()` | Detect current session from `====` or `PIPELINE v3` markers |
| `_parse_checkpoint()` | Extract stats from checkpoint JSON, filters by session timestamp |
| `_get_latest_checkpoint()` | Get most recent checkpoint from current session |
| `_read_logs()` | Read and filter log lines from current session |
| `_update_stats()` | Refresh all dashboard statistics (called every 0.5s) |
| `_build_layout()` | Construct Rich TUI layout with all panels |

## Data Flow

### Initialization Flow

```
__init__(show_logs=True)
  │
  ├─► Set base_dir = ~/ai/knowledge-base
  ├─► Set output_dir = qsys-full-extract
  ├─► _find_latest_log()
  │   └─► Try pipeline.log, then run_*.log (sorted by mtime)
  ├─► session_start (property)
  │   └─► _get_session_start_from_log()
  │       ├─► Scan log for "====" or "PIPELINE v3" markers
  │       ├─► Extract timestamps
  │       └─► Return most recent marker timestamp
  └─► Initialize state variables
```

### Main Loop Flow

```
run()
  │
  └─► Rich Live Display (refresh_per_second=2)
      │
      └─► Loop (every 0.5s):
          ├─► _update_stats()
          │   ├─► _find_process()           # Check if kb_ingest_v3.py running
          │   ├─► _get_latest_checkpoint()  # Parse checkpoint from current session
          │   ├─► _read_logs()              # Get log lines from current session
          │   └─► Update display
          └─► _build_layout()             # Rebuild and refresh TUI
```

### Session Detection Flow

```
_get_session_start_from_log()
  │
  ├─► Read entire log file
  ├─► Find all lines with "=====" or "PIPELINE" + "v3"
  ├─► Extract timestamps from markers
  ├─► Return most recent timestamp
  │
  └─► If no markers found:
      └─► Return first timestamp in log (fallback)
```

### Checkpoint Parsing Flow

```
_parse_checkpoint(checkpoint_path)
  │
  ├─► Read checkpoint JSON file
  ├─► Extract checkpoint.timestamp
  ├─► Compare to session_start
  │   ├─► If timestamp < session_start:
  │   │   └─► Return None (filter out old checkpoint)
  │   └─► Else: continue
  ├─► Parse entities array
  ├─► Count by extraction_model:
  │   ├─► Split ensemble names ("model1,model2" → ["model1", "model2"])
  │   └─► Build model_usage dict
  ├─► Extract chunk number from filename (checkpoint_690.json → 690)
  └─► Return stats dict
```

## Checkpoint Format

The dashboard reads checkpoint files generated by `kb_ingest_v3.py`:

```json
{
  "timestamp": "2026-01-04T23:22:18.566712",
  "config": {
    "source_dir": "/path/to/source",
    "output_dir": "/path/to/output",
    "collection_name": "qsys-lua-v3",
    "chunk_size": 3000,
    "chunk_overlap": 200
  },
  "stats": {
    "chunks": 3427,
    "documents": 690,
    "entities": 8484,
    "relationships": 6038
  },
  "entities": [
    {
      "name": "Timer",
      "type": "COMPONENT",
      "confidence": 0.92,
      "source_chunk": "2c917f9a6856...",
      "extraction_model": "llama3.1:8b,qwen2.5:7b",
      "source_validated": true,
      "type_inferred": false
    }
  ],
  "relationships": [
    {
      "source": "Timer.Start",
      "target": "Timer",
      "type": "BELONGS_TO",
      "confidence": 0.95,
      "extraction_model": "qwen3"
    }
  ]
}
```

### Key Fields Used by Dashboard

| Field | Usage |
|-------|-------|
| `timestamp` | Session filtering (compared to `session_start`) |
| `stats.chunks` | Total chunk count (denominator for progress) |
| `stats.entities` | Total entities count |
| `stats.relationships` | Total relationships count |
| `entities[].extraction_model` | Model name for per-model tracking |

## Session Awareness

### Implementation

```python
def _get_session_start_from_log(self):
    """Get current session start by finding latest pipeline marker."""
    session_starts = []

    for line in log_file:
        if '=====' in line or ('PIPELINE' in line and 'v3' in line):
            match = re.search(r'\[(\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2})', line)
            if match:
                session_starts.append(datetime.fromisoformat(match.group(1)))

    # Return most recent marker
    return session_starts[-1] if session_starts else first_timestamp
```

### Filtering Logic

```python
# Checkpoint filtering
if checkpoint_timestamp < self.session_start:
    return None  # Skip old checkpoint

# Log filtering
if log_timestamp >= self.session_start:
    self.log_lines.append(line)  # Include current session line
```

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| [rich](https://rich.readthedocs.io/) | 13.9.0+ | TUI framework (Live, Panel, Table, Layout) |
| [psutil](https://psutil.readthedocs.io/) | 5.9.0+ | System monitoring (cpu_percent, virtual_memory, disk_usage) |

### Standard Library

| Module | Usage |
|--------|-------|
| `json` | Checkpoint file parsing |
| `re` | Log pattern matching (timestamps, markers) |
| `sys` | Command-line arguments |
| `time` | Sleep delays |
| `pathlib.Path` | File path operations |
| `datetime` | Timestamp parsing and arithmetic |
| `collections.deque` | Fixed-size log buffer |

## Rich TUI Layout

```python
layout["main"].split_row(
    Layout(name="left", ratio=40),   # 60% of screen
    Layout(name="right", ratio=27),   # 40% of screen
)

layout["right"].split_column(
    Layout(name="status", size=15),   # Fixed height
    Layout(name="models", size=10),   # Fixed height
    Layout(name="system", size=5),    # Fixed height
    Layout(name="info", size=5),      # Fixed height
)
```

## Refresh Rate

The dashboard updates at **2 Hz** (2 times per second):

```python
with Live(..., refresh_per_second=2) as live:
    while True:
        self._update_stats()
        live.update(self._build_layout())
        time.sleep(0.5)
```

This provides responsive updates without excessive CPU usage.

## Process Detection

The dashboard detects the running ingestion process by scanning process list:

```python
def _find_process(self):
    for proc in psutil.process_iter(['pid', 'name', 'cmdline']):
        cmdline = proc.info['cmdline']
        if cmdline and any('kb_ingest_v3.py' in str(c) for c in cmdline):
            return True
    return False
```

## Extensibility

### Adding New Panels

1. Create a `_build_<panel>_panel()` method
2. Add panel to layout in `_build_layout()`
3. Update layout ratios/sizes

```python
def _build_custom_panel(self):
    """Build custom panel."""
    grid = Table.grid()
    grid.add_column("Label", style="cyan")
    grid.add_column("Value", justify="right", style="yellow")
    grid.add_row("Custom:", "data")
    return Panel(grid, title="[bold]Custom Panel[/bold]")

# In _build_layout():
layout["right"].split_column(
    ...,
    Layout(name="custom", size=5),
)
layout["custom"].update(self._build_custom_panel())
```

### Custom Data Sources

1. Add path configuration to `__init__()`
2. Create parser method
3. Call parser in `_update_stats()`

```python
def __init__(self, custom_path=None):
    self.custom_path = Path(custom_path) if custom_path else Path.home() / "custom"
    ...

def _parse_custom_data(self):
    # Parse your data file
    ...

def _update_stats(self):
    ...
    self.custom_data = self._parse_custom_data()
```

### Different Session Markers

Modify `_get_session_start_from_log()`:

```python
# Add your pattern
if 'YOUR_SESSION_MARKER' in line:
    # Extract and return timestamp
```

## Performance

| Metric | Value |
|--------|-------|
| Baseline Memory | ~50MB |
| CPU Usage (idle) | ~2% |
| CPU Usage (updating) | ~5% |
| Refresh Rate | 2 Hz (0.5s interval) |
| Log Buffer | 30 lines (filtered) |
| File I/O | Small files (fast) |

## Thread Safety

The dashboard is **single-threaded** by design:
- Main loop runs on primary thread
- File I/O is blocking but fast (small files)
- No race conditions with ingestion pipeline (read-only access)

---

For contribution guidelines, see the main [README.md](README.md).
