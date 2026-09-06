# Dejima


https://github.com/user-attachments/assets/062cb66c-c09c-42f0-8aea-4468ec83fd08


LLM chat in your terminal. Dejima is a fast, keyboard-only chat client for any
**OpenAI-compatible** server you run yourself — Ollama, LM Studio, llama.cpp
server, or similar. No browser, no accounts: just you and the model.

## Features

- **Streaming chat** with per-response telemetry (tokens/s, time-to-first-token)
- **Reasoning display** for models that expose chain-of-thought
- **Sessions**: create, rename, delete, switch, branch — auto-saved between runs
- **Prompt templates**: a personal library of system prompts, edited in `$EDITOR`
- **File context**: attach files to the conversation (`/add`); contents are
  re-read fresh on every send
- **Rich Markdown** rendering: code blocks, tables, lists, quotes
- **History editing** in `$EDITOR`, command completion, filterable popups

## Requirements

- A terminal with TrueColor support
- A running OpenAI-compatible LLM server (local model servers work out of the
  box; **remote services that require API keys are not supported yet** — see
  Troubleshooting)
- Optional: a value for `$EDITOR` (falls back to `vim`)

## Configuration

All files live in `~/.config/dejima/`:

```
~/.config/dejima/
├── models.toml       # required: the models you can chat with
├── config.toml       # optional: default model & prompt
├── prompts/          # one Markdown file per system-prompt template
│   └── rust-expert.md    ← filename (minus .md) is the template name
└── sessions/         # chat history (TOML), managed automatically
```

### models.toml (required)

`endpoint` is the full Chat Completions URL Dejima POSTs to:

```toml
[[model]]
name     = "qwen-32b"
endpoint = "http://localhost:11434/v1/chat/completions"   # Ollama

[[model]]
name     = "local-llama"
endpoint = "http://localhost:1234/v1/chat/completions"    # LM Studio
```

If `models.toml` is missing or empty, Dejima exits at startup with a clear
error — that is intentional.

### config.toml (optional)

```toml
default_model  = "qwen-32b"    # defaults to the first model in models.toml
default_prompt = "rust-expert" # optional system prompt template name
```

## Using Dejima

Type a message, press `Enter`. The reply streams in live. Two `Esc` presses
within one second abort a stream (the partial answer is kept).

### Commands

Type a slash command in the input line; `Tab` / `Shift-Tab` cycle completion.

| Command           | What it does                                                  |
| ----------------- | ------------------------------------------------------------- |
| `/session`        | Sessions popup: create, rename, delete, branch, switch        |
| `/model`          | Choose the active model                                       |
| `/prompt`         | Prompt templates: select, create, edit, rename, delete, clear |
| `/add`            | Attach file(s) to the conversation (file browser popup)       |
| `/drop`           | Remove an attached file                                       |
| `/files`          | Show a summary of attached files in the chat                  |
| `/edit`           | Edit the whole chat history in `$EDITOR`                      |
| `/editor [text]`  | Compose the pending message in `$EDITOR`                      |
| `/copy`           | Copy the last reply to the clipboard                          |
| `/regenerate`     | Discard the last reply and re-generate it                     |
| `/generate-title` | Name the session automatically from the conversation          |
| `/quit`           | Quit                                                          |

### Keyboard

| Keys                     | Action                             |
| ------------------------ | ---------------------------------- |
| `Enter`                  | Send message / run command         |
| `Tab` / `Shift-Tab`      | Cycle command completion           |
| `Ctrl+o`                 | Compose current input in `$EDITOR` |
| `Ctrl-u` / `Ctrl-d`      | Scroll chat up / down              |
| `Esc` `Esc` (within 1 s) | Abort streaming                    |
| `Ctrl+C`                 | Quit                               |

In popups:

| Keys                                       | Action                                                                    |
| ------------------------------------------ | ------------------------------------------------------------------------- |
| `Tab` / `Shift-Tab` or `Ctrl-n` / `Ctrl-p` | Next / previous item                                                      |
| `Ctrl-f`                                   | Filter the list (letters narrow it, `Backspace` edits, `Esc` ends filter) |
| `Enter`                                    | Select / confirm                                                          |
| `Esc`                                      | Close popup                                                               |
| `n` / `r` / `d` / `b`                      | Session popup: new / rename / delete / branch                             |
| `n` / `e` / `r` / `d` / `c`                | Prompt popup: new / edit / rename / delete / clear                        |
| `Space` / `Ctrl-a`                         | File browser: toggle selection / select all                               |
| `Backspace`                                | File browser: go to parent folder                                         |
| `y` / `n`                                  | Confirm or cancel a delete                                                |

### Working with files

`/add` opens a file browser. Navigate with `Enter` (directories) and `Backspace`
(parent), mark several files with `Space` (or `Ctrl-a`), confirm with `Enter`.
Attached file contents are re-read from disk on **every** message, so editing a
file outside Dejima is picked up automatically. `/drop` detaches; `/files` shows
what is attached.

### Sessions

Dejima auto-saves each session once you send your first message. Branching
(`/session` → `b`) clones the current conversation into a new session so you can
explore a different direction.

## Troubleshooting

- **"Failed to load models.toml"** — create `~/.config/dejima/models.toml`
  (see Configuration above). The expected path is printed in the error.
- **HTTP 401 / auth errors from a remote provider** — API-key support is not
  implemented yet; Dejima only talks to endpoints that don't require auth
  (typically local servers).
- **`/edit` or `/editor` opens an unexpected editor** — set `$EDITOR`
  (otherwise `vim` is used).
