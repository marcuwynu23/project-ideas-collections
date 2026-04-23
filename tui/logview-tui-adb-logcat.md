## LogView TUI (ADB Logcat Terminal Viewer)

A terminal-based interactive tool that wraps around Android’s Android Debug Bridge logcat output, adding pause/resume, filtering, and search features.

---

## Core Concept

Instead of running:

    adb logcat

you run:

    logview

and get an interactive terminal interface for viewing logs.

---

## Key Features

### 1. Live Log Streaming

- Continuously reads `adb logcat`
- Displays logs in real time in a scrollable interface
- Color coding by log level:
  - DEBUG
  - INFO
  - WARN
  - ERROR

---

### 2. Pause and Resume

- Press `p` to pause incoming logs
- Press `r` to resume streaming
- Optional buffering while paused

---

### 3. Search Mode

- Press `/` to search inside logs
- Highlights matching text
- Navigate between matches

---

### 4. Filtering System

Support filtering by:

- Log tag (e.g. ActivityManager, System.out)
- Log level (ERROR only, WARN only, etc.)
- Regex-based filtering
- Combined filters (e.g. ERROR AND Wifi)

---

### 5. Bookmarking (Optional)

- Mark important log lines
- View saved logs in a separate section

---

### 6. Split View (Advanced)

- Left panel: live logs
- Right panel: filtered results or search view

---

## Architecture

### 1. Log Collector

- Runs adb logcat -v time
- Reads stdout stream continuously

### 2. Parser

- Parses log structure:
  - timestamp
  - pid
  - tag
  - level
  - message

### 3. Buffer Layer

- Stores logs in memory using a ring buffer

### 4. Filter Engine

- Applies text, regex, tag, and level filters

### 5. TUI Renderer

Recommended options:

- Python: textual or rich
- Rust: ratatui
- Go: tview

---

## Suggested Tech Stack

Beginner-friendly:

- Python
- textual
- subprocess

Performance-focused:

- Rust
- ratatui
- tokio for async handling

---

## Keyboard Controls Example

| Key | Action           |
| --- | ---------------- |
| p   | Pause            |
| r   | Resume           |
| /   | Search           |
| f   | Open filter menu |
| q   | Quit             |
| j/k | Scroll           |

---

## Stretch Goals

- Export logs to file
- Save filter presets
- Multi-device support
- Device selector using adb devices
- Crash detection highlighting (ANR, FATAL EXCEPTION)

---

## Why this project is useful

- Works with real Android system tooling
- Teaches streaming data handling
- Builds strong terminal UI skills
- Good portfolio project for developer tools
