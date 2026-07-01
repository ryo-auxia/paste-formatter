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

## Sending text from another tool

Pass content in via the URL so another tool (a shell script, Raycast, an editor command) can hand text to the formatter:

- **Use the hash fragment**, not a query string: `.../#text=<url-encoded>`. The fragment is never sent to a server, so there is no length cap (a `?text=` query is capped at ~7KB on GitHub Pages — it returns HTTP 414 past that — and would be logged), and the content stays private.
- Optional params (inside the hash, parsed as a query string): `&target=<t>` sets the default target; `&copy=<t>` attempts to copy on load. Targets: `docs`, `slackRich`, `slackMrkdwn`, `sheets`, `plain`, `markdown`.
- Auto-copy on load is best-effort — browsers block clipboard writes without a focused tab / user gesture. The reliable path is: the URL pre-fills the text, then press `Ctrl+1..6`.
- The page clears the fragment from the address bar after reading it.

Example macOS shell function (encodes the clipboard and opens the formatter):

```bash
fmt() {
  local t; if [ -t 0 ]; then t="$(pbpaste)"; else t="$(cat)"; fi
  local enc; enc=$(printf '%s' "$t" | python3 -c 'import sys,urllib.parse;print(urllib.parse.quote(sys.stdin.read()))')
  open "https://ryo-auxia.github.io/paste-formatter/#text=${enc}&target=slackMrkdwn"
}
# usage:  fmt           (from clipboard)
#         cat notes.md | fmt
```

## Implementation

One self-contained `index.html`, vanilla JS, no build step. The only external dependency is [marked](https://github.com/markedjs/marked) loaded from jsdelivr at page load. Markdown is parsed once into a DOM; each copy mode is a serializer over that DOM, written to the clipboard as `text/html` and/or `text/plain` via the async Clipboard API.
