# Paste Formatter

A single-file web tool that converts AI agent (mainly Claude Code) markdown output into clipboard formats optimized for different paste targets, so formatting survives the paste with no manual cleanup.

**Use it:** open `index.html` locally, or the hosted page via GitHub Pages.

## Copy modes

| Mode | What lands on the clipboard |
|---|---|
| Docs / Gmail | Rich HTML: real headings, lists, tables, monospace code (inline styles, which Google Docs keeps) |
| Slack (rich) | HTML adapted for Slack's composer: headings flattened to bold, tables as aligned monospace blocks |
| Slack mrkdwn | Plain text in Slack syntax: `*bold*`, `_italic_`, `•` bullets, ``` blocks |
| Sheets (table) | A markdown table as a real HTML table (pastes as cells) plus TSV fallback; picker for multiple tables |
| Plain text | All markdown syntax stripped, structure kept |
| Markdown (Notion) | Cleaned markdown as-is (Notion parses it natively) |

## Features

- Cleanup pass (default on) strips Claude Code terminal artifacts: ANSI codes, tool-use markers, `42→` line-number prefixes, progress-tree box-drawing characters
- Live rendered preview
- Auto-copy on paste with a configurable default target
- Keyboard shortcuts Ctrl+1..6
- Drag-and-drop `.md` / `.txt` files
- Dark / light theme via `prefers-color-scheme`
- Settings persisted in localStorage

## Implementation

One self-contained `index.html`, vanilla JS, no build step. The only external dependency is [marked](https://github.com/markedjs/marked) loaded from jsdelivr at page load. Markdown is parsed once into a DOM; each copy mode is a serializer over that DOM, written to the clipboard as `text/html` and/or `text/plain` via the async Clipboard API.
