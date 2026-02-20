# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**red4-conflicts** is a native desktop GUI application (Rust + egui) that scans Cyberpunk 2077 `.archive` mod files, detects file-level conflicts between mods, and visualizes which mods "win" or "lose" based on load order. It also supports load order management via `modlist.txt`.

## Build Commands

```bash
cargo build              # Debug build
cargo build --release    # Release build (hides Windows console)
cargo run                # Run the application
cargo clippy             # Lint (project warns on clippy::all + rust_2018_idioms)
cargo fmt                # Format (default rustfmt rules)
```

No tests exist in this project.

## Architecture

Three-file structure:

- **`src/main.rs`** — Native entry point. Sets up eframe window (400x300), file logging (`red4-conflicts.log`), and launches `TemplateApp`. No WASM target despite leftover PWA assets in `assets/`.

- **`src/lib.rs`** — Core data model and business logic.
  - `TemplateApp` — Main app state: game path, parsed archives, conflict map, load order, UI settings. State is persisted via serde + eframe persistence.
  - `ArchiveViewModel` — Represents a single `.archive` with its file hashes and winning/losing status.
  - `generate_conflict_map()` — Core algorithm: iterates archives in reverse load order, determines which archive "wins" each conflicting file (last to load wins).
  - `reload_load_order()` — Scans game folder for `.archive` files, applies binary-alphabetical sort, then overlays `modlist.txt` ordering if present.

- **`src/app.rs`** — UI layer implementing `eframe::App::update()`.
  - **Top panel**: Menu bar (file operations, theme toggle, about/links).
  - **Left panel**: Load order list with optional drag-and-drop reordering (`egui_dnd`).
  - **Central panel**: Archive path selector, filters, scrollable conflict grid with color coding (green=winning, red=losing, gray=fully obsolete). Three visualization modes for conflicts: tooltip, inline, collapsing dropdown.

## Key Dependencies

- **`red4lib`** (git dependency from `oBusk/red4lib`, branch `wolvenkit-kark`) — Cyberpunk archive parsing and hash resolution. Not on crates.io. A commented-out local path in `Cargo.toml` exists for local development.
- **`egui`/`eframe` 0.32** — Immediate-mode GUI framework with `glow` OpenGL backend.
- **`rfd`** — Native file/folder dialogs.
- **`egui_dnd`** — Drag-and-drop list reordering.

## Code Conventions

- Clippy warnings are enforced: `#![warn(clippy::all, rust_2018_idioms)]` in both `main.rs` and `lib.rs`.
- Format on save is configured in `.vscode/settings.json`.
- The app is Windows-focused (hides console in release builds) but the code is cross-platform compatible.
