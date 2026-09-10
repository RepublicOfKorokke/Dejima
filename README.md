# Dejima

https://github.com/user-attachments/assets/062cb66c-c09c-42f0-8aea-4468ec83fd08

LLM chat in your terminal. Dejima is a fast, keyboard-only chat client for any
**OpenAI-compatible** server you run yourself — Ollama, LM Studio, llama.cpp
server, or similar. No browser, no accounts: just you and the model.

## Features

- **Streaming chat** with per-response telemetry (tokens/s, time-to-first-token)
- **Reasoning display** for models that expose chain-of-thought
- **Sessions**: create, rename, delete, switch, branch — auto-named and
  auto-saved between runs
- **Prompt templates**: a personal library of system prompts, edited in `$EDITOR`
- **File context**: attach files to the conversation (`/add`); contents are
  re-read fresh on every send
- **Rich Markdown** rendering: code blocks, tables, lists, quotes
- **Chat scrollbar** for long conversations (shown only when scrollable; scroll
  with `Ctrl-u`/`Ctrl-d`)
- **History editing** in `$EDITOR`, command completion, filterable popups

## Requirements

- A terminal with TrueColor support
- A running OpenAI-compatible LLM server (local model servers work out of the
  box; **remote services that require API keys are not supported yet** — see
  Troubleshooting)
- Optional: a value for `$EDITOR` (falls back to `vim`)

## Run

No installation is needed: use the compiled binary provided with this
repository and start it in a terminal with an interactive TTY:

```bash
./Dejima
```

Configuration is created on demand under `~/.config/dejima/` (see below).

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

Each `[[model]]` block declares one model:

- `name` — display name shown in the model popup; must be unique.
- `endpoint` — full Chat Completions URL for the model server.

The file must contain at least one model; if it is missing or empty, Dejima
exits at startup with a clear error — that is intentional.

### config.toml (optional)

```toml
default_model  = "qwen-32b"    # defaults to the first model in models.toml
default_prompt = "rust-expert" # optional system prompt template name
```

- `default_model` — model name (must match a name in `models.toml`). If
  omitted, the first model is used.
- `default_prompt` — prompt file name (a `.md` file in `prompts/`, without
  extension). If omitted, no system prompt is used by default.

### prompts/

System prompt templates, one Markdown file per prompt; the file name (minus
`.md`) is the template's display name. Example
`~/.config/dejima/prompts/rust-expert.md`:

```markdown
You are an expert Rust developer. Write idiomatic, safe, and efficient Rust
code. Follow best practices including proper error handling with Result and
Option types.
```

Manage them from the `/prompt` popup: create with `n` (name → `$EDITOR`),
edit with `e`, rename with `r`, delete with `d` (confirm `y`), clear the
active prompt with `c`. You can also edit the `.md` files directly.

## Using Dejima

Type a message, press `Enter`. The reply streams in live. Two `Esc` presses
within one second abort a stream (the partial answer is kept). Anything that
doesn't match a slash command is sent as a message.

### Commands

Type a slash command in the input line; `Tab` / `Shift-Tab` (or `Ctrl-n` /
`Ctrl-p`) cycle completion.

| Command           | What it does                                                    |
| ----------------- | --------------------------------------------------------------- |
| `/session`        | Sessions popup: create, rename, delete, branch, switch, preview |
| `/model`          | Choose the active model                                         |
| `/prompt`         | Prompt templates: select, create, edit, rename, delete, clear   |
| `/add`            | Attach file(s) to the conversation (file browser popup)         |
| `/drop`           | Remove an attached file                                         |
| `/files`          | Show a summary of attached files in the chat                    |
| `/edit`           | Edit the whole chat history in `$EDITOR`                        |
| `/preview`        | Read the chat history in `$EDITOR` (read-only; edits ignored)   |
| `/editor [text]`  | Compose the pending message in `$EDITOR`                        |
| `/copy`           | Copy the last reply to the clipboard                            |
| `/regenerate`     | Discard the last reply and re-generate it                       |
| `/generate-title` | Name the session from the whole conversation (one line)         |
| `/quit`           | Quit                                                            |

### Keyboard

| Keys                                       | Action                                       |
| ------------------------------------------ | -------------------------------------------- |
| `Enter`                                    | Send message / run command                   |
| `/`                                        | Start a slash command (completion available) |
| `Tab` / `Shift-Tab` or `Ctrl-n` / `Ctrl-p` | Cycle through command completion             |
| `Esc`                                      | Cancel command completion                    |
| `Ctrl+o`                                   | Compose current input in `$EDITOR`           |
| `Ctrl-u` / `Ctrl-d`                        | Scroll chat up / down (10 rows)              |
| `Esc` `Esc` (within 1 s)                   | Abort streaming                              |
| `Ctrl+C`                                   | Quit                                         |

### In popups

Every popup supports: `Tab` / `Shift-Tab` (or `Ctrl-n` / `Ctrl-p`) to move,
`Ctrl-f` to filter (letters narrow the list, `Backspace` edits, `Esc` ends the
filter), and `Esc` to close (or step back from a name/confirm prompt).

**Session popup** (`/session`)

| Key     | Action                                                      |
| ------- | ----------------------------------------------------------- |
| `Enter` | Switch to selected session                                  |
| `p`     | Preview selected session in the chat (press again to close) |
| `n`     | New session (enter a name; blank = auto-named)              |
| `r`     | Rename selected session                                     |
| `d`     | Delete selected session (confirm with `y` / cancel `n`)     |
| `b`     | Branch (clone) selected session                             |

The preview renders the selected session's real messages in the main chat
area _behind_ the popup, under a `[preview] {name}` banner; it does not
change the active session and clears when the popup closes.

**Prompt popup** (`/prompt`)

| Key     | Action                                                 |
| ------- | ------------------------------------------------------ |
| `Enter` | Select prompt as the active system prompt              |
| `n`     | Create new prompt (name → `$EDITOR`)                   |
| `e`     | Edit selected prompt in `$EDITOR`                      |
| `r`     | Rename selected prompt                                 |
| `d`     | Delete selected prompt (confirm with `y` / cancel `n`) |
| `c`     | Clear the system prompt (chat with none)               |

**Model popup** (`/model`): navigate, filter, `Enter` selects the active model.

**File browser** (`/add`): `Enter` enters directories; `Space` toggles file
selection, `Ctrl-a` selects all, `Enter` adds the selected files, `Backspace`
goes to the parent folder.

**Drop popup** (`/drop`): lists attached files; `Enter` detaches the selected
one.

The session and prompt popups stay open after their list operations (session:
delete, rename, branch; prompt: delete, rename) with a refreshed list and your
cursor/filter kept in place, so you can run several operations back to back.

### Working with files

`/add` opens a file browser. Navigate with `Enter` (directories) and `Backspace`
(parent), mark several files with `Space` (or `Ctrl-a`), confirm with `Enter`.
Attached file contents are re-read from disk on **every** message, so editing a
file outside Dejima is picked up automatically. `/drop` detaches; `/files` shows
what is attached.

### Sessions

Dejima auto-saves each session (a TOML file under `~/.config/dejima/sessions/`)
once you send your first message, and on every send after that. Sessions
created without a name are **auto-named** from that first message — the query's
first 30 characters plus the active system prompt (e.g.
`fix the login bug_coder`) — and can be renamed anytime with `/session` → `r`
or titled with `/generate-title`. Branching (`/session` → `b`) clones the
current conversation into a new session so you can explore a different
direction; a blank branch name is derived the same way from the original
conversation. The session header shows the current name and active prompt.

## Troubleshooting

- **"Failed to load models.toml"** — create `~/.config/dejima/models.toml`
  (see Configuration above). The expected path is printed in the error.
- **HTTP 401 / auth errors from a remote provider** — API-key support is not
  implemented yet; Dejima only talks to endpoints that don't require auth
  (typically local servers).
- **`/edit` or `/editor` opens an unexpected editor** — set `$EDITOR`
  (otherwise `vim` is used).
- **Just want to browse the chat, not change it** — use `/preview`; it opens the
  same conversation in `$EDITOR` but ignores everything you type, so the session
  is never modified.
- **Garbled screen after an external editor** — Dejima clears and redraws the
  terminal automatically; if it persists (e.g. after a crash), quit and run the
  shell's `reset` command to restore your terminal.
