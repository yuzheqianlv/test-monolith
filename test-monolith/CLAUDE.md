# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Rust-based monolith tool project that bundles web pages into single HTML files. The project contains:
- `/monolith/` - Reference implementation (DO NOT MODIFY - read-only for understanding)
- Main project root - Your working directory for new translation features

## Build Commands

### Core Build Commands
```bash
# Build the main project
cargo build --locked

# Build with GUI features
cargo build --locked --bin monolith-gui --features="gui"

# Run tests
cargo test --locked

# Format code
cargo fmt --all

# Lint code
cargo clippy --

# Install locally
cargo install --force --locked --path .
```

### Using Makefile (in monolith/ directory)
```bash
# Build both CLI and GUI
make all

# Build CLI only
make build

# Build GUI only
make build-gui

# Run tests
make test

# Format code
make format

# Lint and fix
make lint

# Install
make install
```

## Architecture Overview

The monolith tool follows a modular architecture:

### Core Modules (in src/):
- **core.rs** - Main processing logic, MonolithOptions, error handling
- **html.rs** - HTML parsing, DOM manipulation, asset embedding
- **css.rs** - CSS processing and embedding
- **js.rs** - JavaScript handling
- **url.rs** - URL resolution and data URL creation
- **session.rs** - HTTP session management and asset retrieval
- **cache.rs** - Asset caching system
- **cookies.rs** - Cookie file parsing and handling
- **lib.rs** - Public API exports
- **main.rs** - CLI entry point with clap argument parsing

### Key Data Structures:
- **MonolithOptions** - Configuration struct for all processing options
- **Session** - Manages HTTP requests, caching, and cookies
- **MonolithOutputFormat** - Enum for HTML/MHTML output formats
- **Cache** - On-disk caching for assets above size threshold

### Processing Flow:
1. CLI argument parsing (main.rs)
2. Session initialization with cache and cookies
3. Document retrieval (URL or STDIN)
4. DOM parsing and asset discovery (html.rs)
5. Asset retrieval and embedding (session.rs)
6. Document serialization and output

## Translation Feature Development

For the translation feature requirements:
- Extract text content from HTML documents before asset embedding
- Integrate translation processing between DOM parsing and asset embedding
- Consider text replacement in both HTML content and CSS text properties
- Use lol_html for efficient HTML text processing (see target/doc/lol_html)

## Dependencies

Key dependencies for understanding:
- **html5ever** - HTML parsing and DOM manipulation
- **cssparser** - CSS parsing
- **reqwest** - HTTP client for asset retrieval
- **redb** - On-disk caching database
- **encoding_rs** - Character encoding handling
- **clap** - CLI argument parsing

## Testing

Tests are organized by module in `tests/`:
- Integration tests with sample HTML files in `tests/_data_/`
- Module-specific unit tests mirror src/ structure
- Run `cargo test --locked` to execute all tests