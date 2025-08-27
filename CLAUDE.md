# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **dmenu-flexipatch** - a highly customizable build of dmenu 5.3 that uses preprocessor directives to enable/disable patches at compile time. Unlike traditional patching approaches, this repository contains both patched and original code, allowing users to selectively enable features through configuration.

## Build Commands

```bash
# Quick build and install (removes config files, rebuilds, installs)
./install.sh

# Clean build artifacts
make clean

# Build dmenu with current patch configuration
make

# Clean build
make clean && make

# Install to system (requires appropriate permissions)
sudo make install

# Create distribution tarball
make dist
```

## Repository Structure

### Key Files
- `patches.def.h` - Default patch configuration (copy to `patches.h` before building)
- `patches.h` - Active patch configuration (generated from patches.def.h if missing)
- `config.def.h` - Default dmenu configuration (copy to `config.h` before building) 
- `config.h` - Active runtime configuration
- `config.mk` - Build configuration (compiler flags, optional libraries)
- `dmenu.c` - Main program with conditional patch code
- `patch/` - Directory containing all patch implementations

### Branch Strategy
- `master` - Main branch for upstream compatibility
- `prod` - Production customizations

## Patch Configuration

To enable/disable patches:
1. Edit `patches.h` (or `patches.def.h` if patches.h doesn't exist)
2. Change patch values from 0 (disabled) to 1 (enabled)
3. Run `make clean && make` or `./install.sh`

Example:
```c
#define ALPHA_PATCH 1        // Enable transparency
#define FUZZYMATCH_PATCH 0   // Disable fuzzy matching
```

## Architecture

### Core Components
- `dmenu.c` - Main program logic with conditional compilation based on enabled patches
- `drw.c/drw.h` - Drawing library for X11 rendering
- `util.c/util.h` - Utility functions
- `config.h` - Runtime configuration (colors, fonts, etc.)
- `patches.h` - Compile-time patch selection

### Patch System
- All patches reside in `patch/` directory
- Each patch typically has:
  - `.c` file with implementation
  - `.h` file with declarations (for patches requiring headers)
- Patches are included conditionally using `#if PATCH_NAME` preprocessor directives
- Incompatible patches are handled with override logic at the top of dmenu.c

### Key Architectural Decisions
- Heavy use of preprocessor directives (`#if`, `#ifdef`) throughout the codebase
- Patches modify data structures, enums, and functions conditionally
- Some patches require additional libraries (e.g., ALPHA_PATCH needs -lXrender in config.mk)
- Code blocks wrapped in `#if PATCH_NAME_PATCH` directives for conditional compilation

## Development Workflow

### Adding/Modifying Patches
1. Edit `patches.h` to enable/disable patches
2. Modify patch code in `patch/` directory if needed
3. Follow existing preprocessor directive patterns
4. Test with `./install.sh` or `make clean && make`

### Updating from Upstream
If tracking upstream dmenu-flexipatch:
1. Add upstream remote: `git remote add upstream https://github.com/bakkeby/dmenu-flexipatch.git`
2. Fetch and merge/rebase as appropriate for your workflow

## Important Notes

### When Adding New Features
- Check patch compatibility - some patches conflict with others (see overrides in dmenu.c:21-25)
- If a patch requires external libraries, update config.mk accordingly
- Test with various patch combinations to ensure compatibility

### Common Patch Dependencies
- GRIDNAV_PATCH requires GRID_PATCH
- Highlight patches work differently when combined with FUZZYMATCH_PATCH
- MULTI_SELECTION_PATCH disables several other patches due to incompatibility

### Library Dependencies
- ALPHA_PATCH: Uncomment XRENDER in config.mk
- PANGO_PATCH: Uncomment PANGO sections in config.mk

## Code Style
- **Language**: C99 standard
- **Indentation**: Tabs (not spaces)
- **Naming**: snake_case for functions/variables, SCREAMING_SNAKE_CASE for macros
- **Braces**: K&R style
- **Error handling**: Use `die()` for fatal errors
- **Memory**: Free all allocated resources