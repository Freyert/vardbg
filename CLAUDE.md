# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

vardbg is a Python debugger and profiler that generates animated visualizations of program flow. It helps with learning algorithms by visualizing variable changes, execution times, and program flow. The project uses a tracer-based approach to intercept Python execution and track variable state changes.

**Requires Python 3.8+** (originally supported 3.6+, but updated dependencies require 3.8+).

## Development Setup

```bash
# Clone and setup
git clone https://github.com/CCExtractor/vardbg
cd vardbg

# Using uv (recommended)
uv sync --extra dev

# Or using traditional venv
python3 -m venv venv
source venv/bin/activate
pip install -e .
```

## Common Commands

### Running vardbg
```bash
# Using uv (recommended)
uv run vardbg run <file.py> <function_name> [options]

# Example: Debug quick_sort with arguments
uv run vardbg run sort.py quick_sort -o qsort.json -a 9 -a 3 -a 5 -a 1

# Replay a recorded session
uv run vardbg replay qsort.json -v sort_vis.mp4

# Direct execution (for development)
uv run python debug.py
```

### Testing
```bash
# Run test algorithms
uv run python tests/algos/test_sorting.py
uv run python tests/algos/test_searching.py
uv run python tests/algos/test_misc.py
```

### Code Quality
```bash
# Install pre-commit hooks (highly recommended)
uv run pre-commit install

# Run formatters manually
uv run isort .
uv run black .
```

## Architecture

### Core Components (Mixin-Based Design)

The `Debugger` class in `vardbg/debugger.py` is composed of multiple mixins:
- **Tracer** (`tracer.py`): Intercepts Python execution using `sys.settrace()`, manages frame-by-frame execution tracking, handles stdout redirection, and filters out stdlib code
- **DiffProcessor** (`diff_processor.py`): Processes variable changes using `dictdiffer`, tracks variable history, handles container modifications (lists/dicts/sets)
- **Profiler** (`profiler.py`): Times line execution for performance analysis
- **Replayer** (`replayer.py`): Loads and replays JSON session recordings

This mixin architecture allows the debugger to combine tracing, diffing, profiling, and replay capabilities while keeping concerns separated.

### Data Flow

1. **Execution Capture**: `Tracer.trace_callback()` is invoked by Python for each frame execution (`call`, `line`, `return` events)
2. **Variable Tracking**: `DiffProcessor` diffs locals between frames to detect variable changes, additions, and deletions
3. **Output Writing**: Changes are written to output writers via `OutputDelegate` which broadcasts to:
   - `ConsoleWriter`: Real-time terminal output
   - `JsonWriter`: Session recording for replay
   - `VideoWriter`: Animated visualization generation

### Key Design Patterns

**Frame Scoping**: Each stack frame gets a `FrameScope` object that tracks:
- Previous and current frame info
- Previous and current local variables
- Used to diff variable changes between execution points

**Variable References**: Special comments control variable handling:
- `# vardbg: ignore` - Don't track this variable
- `# vardbg: ref lst[i]` - Track `i` as an index into container `lst` (shown in videos)

**Output Delegation**: All output goes through `OutputDelegate` which manages multiple concurrent writers, allowing simultaneous console output, JSON recording, and video generation.

### Video Generation

Located in `vardbg/output/video_writer/`:
- **Renderer** (`renderer.py`): Draws visualization frames showing code, variables, execution times, and relationships
- **Encoders**: Support for MP4 (OpenCV), GIF, and WebP formats
- **Config**: TOML-based configuration system with overlay support (see `default_config.toml`)

Video generation is CPU-intensive and inflates profiler results, so it's recommended to generate videos from replay rather than during initial execution.

## Code Style

- **Line length**: 120 characters (not the PEP-8 default of 80)
- **Formatting**: Black code style (enforced via `black` tool)
- **Imports**: Organized in 3 sections (stdlib, third-party, project) with `isort`
- **Strings**: Use f-strings (not `%` or `.format()` unless necessary)
- **Naming**: `PascalCase` for classes, `snake_case` for everything else

The code style is enforced by pre-commit hooks. Run `pre-commit install` to set them up.

## Testing Approach

The project uses sample algorithms in `tests/algos/` for testing:
- Sorting algorithms (bubble, insertion, merge, selection, shell)
- Searching algorithms (linear, binary)
- Miscellaneous examples (fibonacci, factorial)

Tests verify that the debugger correctly tracks and visualizes algorithm execution.
