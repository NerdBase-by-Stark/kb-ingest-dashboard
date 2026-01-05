# Knowledge Base Ingest Dashboard v1.0

> A real-time terminal dashboard for monitoring knowledge base ingestion progress

[![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.0.0-brightgreen.svg)](CHANGELOG.md)

## What It Does

The KB Ingest Dashboard provides **live, session-aware monitoring** of document ingestion pipelines. Track your knowledge base construction in real-time from your terminal.

- **Progress Tracking**: Chunks processed with percentage and ETA
- **Entity Extraction**: Real-time counts of extracted entities and relationships
- **LLM Usage**: Per-model API call and token estimates
- **System Resources**: CPU, memory, and disk usage monitoring
- **Live Logs**: Color-coded ingestion logs (optional)
- **Session-Aware**: Only tracks current session, not historical data

## Quick Start

```bash
# Install dependencies
pip install rich psutil

# Download the script
curl -o ~/.local/bin/kb-ingest-dashboard https://raw.githubusercontent.com/your-repo/kb-ingest-dashboard/v1.0/kb-ingest-dashboard
chmod +x ~/.local/bin/kb-ingest-dashboard

# Run the dashboard
kb-ingest-dashboard
```

## Features

| Feature | Description |
|---------|-------------|
| **Session-Aware Tracking** | Detects current session from log markers, filters out historical data |
| **Real-Time Progress** | Live chunk count with progress bar and ETA calculation |
| **Per-Model LLM Stats** | Entity counts, API calls, and token estimates per model |
| **System Monitoring** | CPU, memory, and disk usage with visual bars |
| **Live Logs** | Color-coded log panel (can be hidden) |
| **Auto-Install** | Dependencies install automatically on first run |

## Dashboard Preview

<img width="1143" height="653" alt="image" src="https://github.com/user-attachments/assets/cc4845e0-7d6d-48ab-aec7-d64d08f97dcd" />


## Documentation

- [Installation Guide](INSTALL.md)
- [Usage Guide](USAGE.md)
- [Architecture](ARCHITECTURE.md)
- [Changelog](CHANGELOG.md)

## Requirements

| Requirement | Minimum | Recommended |
|-------------|----------|--------------|
| Python | 3.9+ | 3.12+ |
| Terminal | ANSI color support | iTerm2, GNOME Terminal, Windows Terminal |
| RAM | 50MB | 100MB |
| Disk | 10MB (script only) | Depends on dataset size |

### Python Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| [rich](https://rich.readthedocs.io/) | 13.9.0+ | Terminal UI framework |
| [psutil](https://psutil.readthedocs.io/) | 5.9.0+ | System resource monitoring |

## Installation

See [INSTALL.md](INSTALL.md) for detailed installation instructions, including:
- Direct download method
- Git clone method
- Manual dependency installation
- Troubleshooting

## Usage

```bash
# Show all panels (default)
kb-ingest-dashboard

# Hide log panel (cleaner view)
kb-ingest-dashboard --hide-logs
kb-ingest-dashboard --no-logs  # Alias
```

See [USAGE.md](USAGE.md) for:
- Panel descriptions
- Session awareness explanation
- Workflows and use cases
- Interpreting the display
- Tips and best practices

## How It Works

The dashboard works alongside your ingestion pipeline (`kb_ingest_v3.py`):

1. **Detects session start** by reading log markers (`====` or `PIPELINE v3`)
2. **Parses checkpoint files** (`checkpoint_*.json`) for progress and model usage
3. **Reads live logs** for real-time activity display
4. **Monitors system resources** via psutil
5. **Updates every 0.5 seconds** (2 Hz refresh rate)

### Session Awareness

The dashboard is **session-aware** - it only tracks data from the current ingestion session:

- ✅ Counts entities extracted in current session only
- ✅ Shows progress from session start (not cumulative)
- ✅ Ignores checkpoints from previous sessions
- ✅ Calculates ETA based on current session timing

Without session awareness, you'd see inflated counts from all historical runs.

## Development

See [ARCHITECTURE.md](ARCHITECTURE.md) for:
- Technical architecture
- Class structure
- Data flow diagrams
- Extension guide

## License

MIT License - see [LICENSE](LICENSE)

## Contributing

Contributions welcome! Please feel free to submit a Pull Request.

## Version History

See [CHANGELOG.md](CHANGELOG.md) for version history.

## Acknowledgments

Built with:
- [Rich](https://rich.readthedocs.io/) - Beautiful terminal output
- [psutil](https://psutil.readthedocs.io/) - System monitoring

---

**KB Ingest Dashboard v1.0** - Monitor your knowledge base ingestion with style.
