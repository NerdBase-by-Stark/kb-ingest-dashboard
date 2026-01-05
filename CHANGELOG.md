# Changelog

All notable changes to the KB Ingest Dashboard will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-01-05

### Added
- **Initial release** of KB Ingest Dashboard v1.0
- Real-time TUI with Rich library (2 Hz refresh rate)
- **Session-aware tracking** - filters data by session start markers
- Progress panel with chunk count, percentage, progress bar, and ETA
- Ingestion Status panel with detailed metrics (elapsed, rate, entities, relationships, LLM usage)
- **LLM Usage panel** with per-model tracking:
  - Entity counts per model
  - Estimated API calls per model
  - Estimated token usage per model
- System Resources panel (CPU, memory, disk) with visual mini bars
- Live log panel with color-coded output (cyan, green, magenta)
- Info panel with session details and controls
- Auto-detection of running `kb_ingest_v3.py` process
- Checkpoint parsing from `checkpoint_*.json` files
- Session markers detection (`====` or `PIPELINE v3`)
- `--hide-logs` and `--no-logs` options for cleaner view
- Auto-installation of dependencies on first run
- Right-aligned System Resources labels with gap

### Technical Details
- Layout ratio: 40:27 (left:right) for wider right panel
- Panel heights: Status=15, LLM Usage=10, System=5, Info=5
- Model column: auto-width (takes remaining space)
- Entities column: width=7
- Est. LLM Calls column: width=9
- Est. Tokens column: width=11
- Token estimate: ~750 tokens per call (based on ~3000 char chunks)

### Session Tracking
- Detects current session from LATEST `====` or `PIPELINE v3` marker
- Filters checkpoints by timestamp (excludes checkpoints older than session start)
- Filters logs to only show entries from current session
- Uses session_start for all timing calculations (elapsed, ETA)

### Model Parsing
- Parses actual model usage from checkpoint `extraction_model` field
- Splits ensemble models (e.g., `llama3.1:8b,qwen2.5:7b`) for accurate per-model counts
- Shows only models with entities in current session
- Fallback to all models if no checkpoint data available

## [Unreleased]

### Planned
- Configurable paths via environment variables
- Multiple ingestion job monitoring
- Export stats to JSON/CSV
- Historical comparison mode
- `--debug` flag for troubleshooting
- `--version` flag for version display

---

## Version Summary

| Version | Date | Key Changes |
|---------|------|-------------|
| v1.0.0 | 2026-01-05 | Initial official release |
| v0.98.4 | 2026-01-05 | Optimized column widths for model names |
| v0.98.3 | 2026-01-05 | Added gap between labels and bars in System Resources |
| v0.98.2 | 2026-01-05 | Increased right panel width, fixed System Resources alignment |
| v0.98.1 | 2026-01-05 | Added Entities column to LLM Usage panel |
| v0.98 | 2026-01-05 | Session tracking implementation |
