# Installation Guide

## System Requirements

| Requirement | Minimum | Recommended |
|-------------|----------|--------------|
| **Operating System** | Linux, macOS, Windows (WSL) | Ubuntu 22.04+, macOS 13+, Windows 11 + WSL2 |
| **Python** | 3.9 | 3.12+ |
| **Terminal** | ANSI color support, Unicode | iTerm2, GNOME Terminal, Windows Terminal, Alacritty |
| **RAM** | 50MB for dashboard | 100MB+ |
| **Disk** | ~10MB for script | Depends on dataset size (often GBs for KB) |
| **Permissions** | Write access to ~/.local/bin | Write access to installation directory |

> **Note**: The dashboard is designed for ARM64 systems (like NVIDIA Grace, Apple Silicon) and x86_64.

## Installation Methods

### Method 1: Direct Download (Recommended)

The simplest way to install:

```bash
# Download to ~/.local/bin/
curl -o ~/.local/bin/kb-ingest-dashboard https://raw.githubusercontent.com/your-repo/kb-ingest-dashboard/main/kb-ingest-dashboard

# Or using wget
wget -O ~/.local/bin/kb-ingest-dashboard https://raw.githubusercontent.com/your-repo/kb-ingest-dashboard/main/kb-ingest-dashboard

# Make executable
chmod +x ~/.local/bin/kb-ingest-dashboard

# Verify installation
kb-ingest-dashboard --help
```

### Method 2: Clone from Git

```bash
# Clone repository
git clone https://github.com/your-repo/kb-ingest-dashboard.git
cd kb-ingest-dashboard

# Copy to your bin directory
cp kb-ingest-dashboard ~/.local/bin/
chmod +x ~/.local/bin/kb-ingest-dashboard

# Verify
kb-ingest-dashboard --version
```

### Method 3: Manual Download

1. Download the script from [GitHub Releases](https://github.com/your-repo/kb-ingest-dashboard/releases)
2. Save to `~/.local/bin/kb-ingest-dashboard`
3. Make it executable:

```bash
chmod +x ~/.local/bin/kb-ingest-dashboard
```

## Dependency Installation

### Automatic Installation

The dashboard will **automatically install** missing dependencies on first run:

```bash
kb-ingest-dashboard
# If rich or psutil is missing, it will run:
# pip install rich psutil
```

### Manual Installation

To install dependencies manually (recommended for systems without internet):

```bash
# Using pip
pip install rich psutil

# Using conda
conda install -c conda-forge rich psutil

# Using uv (faster alternative to pip)
uv pip install rich psutil

# For ARM64 systems (NVIDIA Grace, Apple Silicon)
conda install -c conda-forge rich psutil
```

### Dependency Versions

| Package | Minimum Version | Install Command |
|---------|-----------------|-----------------|
| rich | 13.9.0 | `pip install "rich>=13.9.0"` |
| psutil | 5.9.0 | `pip install "psutil>=5.9.0"` |

### requirements.txt

Create a `requirements.txt` file:

```
rich>=13.9.0
psutil>=5.9.0
```

Then install with:

```bash
pip install -r requirements.txt
```

## Directory Structure

The dashboard expects this default directory structure:

```
~/ai/knowledge-base/
├── qsys-full-extract/          # Default output directory
│   ├── pipeline.log            # Live ingestion log
│   ├── checkpoint_*.json       # Progress checkpoints (every 10 chunks)
│   ├── checkpoint_10.json
│   ├── checkpoint_20.json
│   └── ...
└── scripts/
    └── kb_ingest_v3.py         # Ingestion pipeline
```

### Custom Paths

To use a different directory structure, edit these lines in the script:

```python
# Lines ~46-47 in kb-ingest-dashboard
self.base_dir = Path.home() / "ai" / "knowledge-base"
self.output_dir = self.base_dir / "qsys-full-extract"
```

## Verification

Test your installation:

```bash
# Check version
head -5 ~/.local/bin/kb-ingest-dashboard
# Should show: KB Ingest Dashboard v1.0

# Check executable bit
ls -la ~/.local/bin/kb-ingest-dashboard
# Should show: -rwxr-xr-x (executable)

# Test run (will show error if no data, but confirms dependencies)
kb-ingest-dashboard
```

## Troubleshooting

### Issue: "Permission denied"

```bash
# Make script executable
chmod +x ~/.local/bin/kb-ingest-dashboard

# If ~/.local/bin doesn't exist, create it first:
mkdir -p ~/.local/bin
chmod +x ~/.local/bin/kb-ingest-dashboard
```

### Issue: "ModuleNotFoundError: No module named 'rich'"

```bash
# Install dependencies manually
pip install rich psutil

# Or using conda
conda install -c conda-forge rich psutil

# Verify installation
python3 -c "import rich; print('Rich OK')"
python3 -c "import psutil; print('Psutil OK')"
```

### Issue: "Command not found: kb-ingest-dashboard"

```bash
# Check if ~/.local/bin is in your PATH
echo $PATH | grep -o ~/.local/bin

# If not shown, add to your shell config:
# For bash (add to ~/.bashrc):
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# For zsh (add to ~/.zshrc):
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### Issue: Display looks wrong/garbled

**Problem**: Terminal doesn't support Unicode or ANSI colors.

**Solution**: Use a modern terminal:
- **macOS**: iTerm2, Terminal.app
- **Linux**: GNOME Terminal, Konsole, Alacritty
- **Windows**: Windows Terminal, Git Bash (NOT cmd.exe)

### Issue: "No such file or directory: pipeline.log"

**Problem**: Dashboard can't find the log file.

**Solution**:
1. Ensure you're running the ingestion pipeline first
2. Check the output directory path in the script
3. Create a symlink if using a different structure:

```bash
# Create symlink if needed
ln -s /your/actual/path ~/ai/knowledge-base/qsys-full-extract
```

### Issue: "All models showing 0 entities"

**Problem**: Checkpoint files aren't being read or are from a previous session.

**Solution**:
1. Check that checkpoints exist:
```bash
ls ~/ai/knowledge-base/qsys-full-extract/checkpoint_*.json
```

2. Check checkpoint timestamp is after session start (the dashboard filters old checkpoints)

3. Restart the ingestion pipeline to generate fresh checkpoints

### Issue: Dashboard shows old/historical data

**Problem**: Session detection isn't working.

**Solution**:
1. Check your log file has session markers:
```bash
grep -E "=====|PIPELINE v3" ~/ai/knowledge-base/qsys-full-extract/pipeline.log
```

2. If no markers found, the dashboard will use the first timestamp (may be old)

3. To force a new session, restart your ingestion pipeline

## Platform-Specific Notes

### NVIDIA DGX / Grace Hopper (ARM64)

```bash
# Use conda-forge for best ARM64 support
conda install -c conda-forge rich psutil

# Or use miniforge if you don't have conda
~/miniforge3/bin/conda install -c conda-forge rich psutil
```

### Apple Silicon (M1/M2/M3)

```bash
# No special steps needed - standard installation works
pip install rich psutil
```

### Windows (WSL2)

```bash
# Install in WSL2 Ubuntu
pip install rich psutil

# Use Windows Terminal for best results
# https://aka.ms/terminal
```

## Uninstallation

```bash
# Remove the script
rm ~/.local/bin/kb-ingest-dashboard

# Optionally remove dependencies
pip uninstall rich psutil
```

## Upgrading

```bash
# Backup current version
cp ~/.local/bin/kb-ingest-dashboard ~/.local/bin/kb-ingest-dashboard.backup

# Download new version
curl -o ~/.local/bin/kb-ingest-dashboard https://raw.githubusercontent.com/your-repo/kb-ingest-dashboard/main/kb-ingest-dashboard

# Make executable
chmod +x ~/.local/bin/kb-ingest-dashboard
```

## Next Steps

After installation:

1. Read the [Usage Guide](USAGE.md)
2. Start your ingestion pipeline
3. Run the dashboard in a separate terminal
4. Monitor your knowledge base construction in real-time!

---

Still having issues? Please [open an issue](https://github.com/your-repo/kb-ingest-dashboard/issues) with:
- Your OS and Python version
- The exact error message
- Steps to reproduce the problem
