# Dejima

A fast, privacy-first terminal (TUI) LLM chat client with agentic tools, local session management, and zero telemetry.

https://github.com/user-attachments/assets/79bed8be-ec23-44ba-8027-ba5f7b0fcba0

## Features

| Area | What you get |
| :--- | :--- |
| **Sessions** | Create, rename, delete, branch, switch, and preview chats; auto-save; title generation. |
| **Chat** | Streaming Markdown (code, tables, quotes), chain-of-thought display, and per-response tokens/s plus time to first token (TTFT). |
| **Models** | OpenAI-compatible endpoints, native Google Gemini, and OpenCode Go; per-model API keys; selectable hyperparameters. |
| **Agent mode** | `bash`, `read_file`, `write_file`, `edit_file`, and subagents in a loop; per-tool `ask`/`auto`/`deny`; diff previews; typed-`YES` guardrails. |
| **Input & customization** | Prompt templates, message snippets, `$EDITOR` history editing, post-response and approval hooks, remappable keys. |
| **Privacy** | No telemetry, no background requests, and plain-text sessions that stay on your machine. |

## Requirements

| Component | Requirement |
| :--- | :--- |
| **Terminal** | ANSI color support; TrueColor recommended. |
| **LLM endpoint** | Any OpenAI-compatible endpoint, native Google Gemini API, or OpenCode Go. |

## Quick Start

### 1. Download

Download the compiled binary from the [releases](https://github.com/RepublicOfKorokke/Dejima/releases) page. No installation is needed.

> **macOS:** the binary is not notarized. A browser download (Safari, Chrome) gets a quarantine flag, and macOS may refuse to launch it. Clear the flag once:

```bash
xattr -d com.apple.quarantine ./dejima
```

> Downloading with `curl` or `wget` avoids the quarantine flag entirely.

`dejima --help` prints every flag with its default and the `DEJIMA_ROOT` variable; `dejima --version` prints the version. Both exit before any configuration is read, so they work on a fresh install.

### 2. Configure a model

Create the config directory and a `models.toml`:

```bash
mkdir -p ~/.config/dejima
```

```toml
# ~/.config/dejima/models.toml
[[model]]
name     = "qwen-32b"
endpoint = "http://localhost:11434/v1/chat/completions"

# For a hosted endpoint, uncomment this and export the variable:
# api_key_env = "OPENAI_API_KEY"
```

See [models.toml](#modelstoml-required) for every field and for Gemini and OpenCode Go examples.

### 3. Run

```bash
./dejima
```

Type a message and press `Enter`. The reply streams in. Two `Esc` presses within one second cancel a stream or a running agent turn. `Ctrl-k` opens the command palette while composing.

## Usage

### Commands

Type `/` at the start of the input to start a slash command, or press `Ctrl-k` while composing to pick one from the palette.

| Group | Command | What it does |
| :--- | :--- | :--- |
| Sessions | `/session` | Open the sessions popup: create, rename, delete, branch, switch, preview. |
| Sessions | `/timeline` | Jump to a User or Assistant message in the current session. |
| Models | `/model` | Choose the active model. |
| Models | `/hyper-params` | Set generation hyperparameters; the value popup's `c` clears the selection. |
| Input | `/prompt` | Select or clear a system prompt template. |
| Input | `/snippet` | Send a saved message snippet (no arguments). |
| Input | `/editor [text]` | Compose the pending message in `$EDITOR`. |
| Input | `/edit` | Edit the whole chat history in `$EDITOR`. |
| Chat | `/preview` | Read the chat history in `preview_command` (read-only; falls back to the editor). |
| Chat | `/copy` | Copy the last reply to the clipboard. |
| Chat | `/regenerate` | Discard and regenerate the last reply (archived to `{id}.trash.toml`). |
| Chat | `/generate-title` | Name the session from the conversation. |
| Agent | `/agent` | Toggle agent mode on or off. |
| Agent | `/subagent <name> [request]` | Delegate to a subagent; bare `/subagent` opens the roster. |
| App | `/quit` | Quit Dejima. |

Commands that take no arguments reject trailing text (for example `/prompt foo`); `/editor [text]` and `/subagent <name> [request]` accept it. A message whose text only starts with a command name — with no separating space, such as `/snippets are useful` — is sent normally.

The text you were typing is kept when a command opens a popup; `/editor` opens that text in `$EDITOR`, and cancelling the editor restores it.

`/timeline` lists User and Assistant messages only. Rows show a fixed 1-based index and use the chat sender colours; `Enter` jumps to the selected message, `Esc` closes, `Ctrl-f` filters, and `Tab` / `Shift-Tab` (or `Ctrl-n` / `Ctrl-p`) navigate.

### Keybindings

All 34 actions are remappable under `[keys]` in `config.toml`; free-text typing is not. Bindings resolve per context, so the same key can mean different things on the main screen and inside a popup.

| Context | Key | Action |
| :--- | :--- | :--- |
| **Main** | `Enter` | Send the message, or run the slash command. |
| **Main** | `Ctrl-o` | Compose the current input in `$EDITOR`. |
| **Main** | `Ctrl-k` | Open the command palette (while composing; no-op while busy or in a popup). |
| **Main** | `Ctrl-u` / `Ctrl-d` | Scroll the transcript up / down. |
| **Main** | `Esc` `Esc` | Cancel a stream or a running agent turn. |
| **Main** | `Ctrl-c` | Clear the input; quit when the input is empty or a popup is open. |
| **List popups** | `Tab` / `Shift-Tab` | Next / previous item (`Ctrl-n` / `Ctrl-p`). |
| **List popups** | `Ctrl-f` | Start filtering (clears the query). |
| **List popups** | `Esc` | End filtering if active; otherwise close the popup. |
| **Confirm** | `y` / `n` | Approve / deny. |
| **Confirm** | `Tab` | Toggle between the diff preview and the chat. |
| **Text entry** | `Enter` / `Esc` | Confirm / cancel the typed name. |

<details>
<summary><strong>Full keybinding reference (all 34 actions, by context)</strong></summary>

#### Main input

| Action id | Default keys | Action |
| :--- | :--- | :--- |
| `submit_message` | `Enter` | Send the message, or run the slash command. |
| `open_command_palette` | `Ctrl-k` | Open the command palette over the draft. Does nothing while busy or while a popup is open; `Esc` closes the palette. |
| `cancel` | `Esc` | Close the inline `/` completion. While busy, the first press arms the double-`Esc` cancel window. |
| `open_editor` | `Ctrl-o` | Compose the current input in `$EDITOR`; the saved text replaces the input. |
| `quit` | `Ctrl-c` | Clear the input (and its completion); quit when the input is empty or a popup is open. |
| `delete_char` | `Backspace` | Delete the last input character. Busy is read-only. |
| `scroll_up` | `Ctrl-u` | Scroll the transcript up by `scroll_step` rows (default 10). |
| `scroll_down` | `Ctrl-d` | Scroll the transcript down by `scroll_step` rows. |
| `completion_next` | `Tab`, `Ctrl-n` | Cycle the inline `/` completion forward (wraps). |
| `completion_prev` | `Shift-Tab`, `Ctrl-p` | Cycle the inline `/` completion backward (wraps). |

#### Shared list popups

These bindings work in `/session`, `/model`, `/prompt`, `/snippet`, `/hyper-params`, `/timeline`, `/subagent`, and the command palette.

| Action id | Default keys | Action |
| :--- | :--- | :--- |
| `list_next` | `Tab`, `Ctrl-n` | Move the selection down (wraps). |
| `list_prev` | `Shift-Tab`, `Ctrl-p` | Move the selection up (wraps). |
| `filter_toggle` | `Ctrl-f` | Start filtering and clear the query. The command palette is always filtering, so there it only clears the query. |
| `list_close` | `Esc` | End filtering if active, otherwise close the popup. The command palette has no separate filtering stage, so `Esc` closes it on the first press. |

Typed characters narrow the list once filtering is active (always active in the command palette).

#### Command palette

| Action id | Default keys | Action |
| :--- | :--- | :--- |
| `command_run` | `Enter` | Run the selected command. The draft is kept; `/editor` opens it in `$EDITOR`, and `/quit` skips the restore. An empty filter does nothing. |

#### Session popup (`/session`)

| Action id | Default keys | Action |
| :--- | :--- | :--- |
| `session_open` | `Enter` | Switch to the selected session. |
| `session_new` | `n` | Create a session (name entry). |
| `session_rename` | `r` | Rename the selected session. |
| `session_delete` | `d` | Delete the selected session (confirmation). |
| `session_branch` | `b` | Branch the selected session into a new one. |
| `session_preview` | `p` | Toggle the selected session's transcript behind the popup. |

#### Prompt popup (`/prompt`)

| Action id | Default keys | Action |
| :--- | :--- | :--- |
| `prompt_select` | `Enter` | Apply the selected template to the session. |
| `prompt_clear` | `c` | Clear the session's prompt. |

#### Snippet popup (`/snippet`)

| Action id | Default keys | Action |
| :--- | :--- | :--- |
| `snippet_select` | `Enter` | Send the selected snippet's content as a user message. |

#### Model popup (`/model`)

| Action id | Default keys | Action |
| :--- | :--- | :--- |
| `model_select` | `Enter` | Make the selected model active. |

#### Hyper-parameters popup (`/hyper-params`)

| Action id | Default keys | Action |
| :--- | :--- | :--- |
| `hyperparam_open` | `Enter` | Open the selected parameter's value list; in the value list, apply the selected value. |
| `hyperparam_clear` | `c` | Clear the session's value and fall back to the model default (value list only). |

`Esc` in the value list returns to the parameter list; at the parameter list it closes the popup.

#### Timeline popup (`/timeline`)

| Action id | Default keys | Action |
| :--- | :--- | :--- |
| `timeline_jump` | `Enter` | Jump the transcript to the selected message. |

#### Subagent popup (`/subagent`)

| Action id | Default keys | Action |
| :--- | :--- | :--- |
| `subagent_complete` | `Enter` | Complete `/subagent <name> ` into the input, ready for the request. |

#### Confirmation dialogs

| Action id | Default keys | Action |
| :--- | :--- | :--- |
| `confirm_yes` | `y` | Confirm the destructive action, or approve the parked tool call. |
| `confirm_no` | `n`, `Esc` | Cancel the action, or deny the tool call. |
| `toggle_confirm_view` | `Tab` | Switch the pending tool approval between the diff preview and the transcript. |

The tool-approval dialog also accepts `Enter` (approve) and `Esc` (deny) literally, even if those actions are remapped; `Ctrl-u` / `Ctrl-d` scroll the pane being shown. Dangerous calls require typing `YES`.

#### Text entry (session names)

| Action id | Default keys | Action |
| :--- | :--- | :--- |
| `text_submit` | `Enter` | Confirm the typed name. |
| `text_cancel` | `Esc` | Cancel the entry without saving. |

Typed characters edit the field; `Backspace` deletes.

</details>

## Agent Mode

Run `/agent` to let the model call tools in a loop until it replies in plain text. It works with OpenAI-compatible and native Gemini models. The tools are `bash`, `read_file`, `write_file`, `edit_file`, and — when you have defined subagents — `subagent`. File tools are confined to the workspace root; `bash` is not sandboxed.

Every tool defaults to `ask`; configure `ask` / `auto` / `deny` per tool in `config.toml` under `[agent]`. When a call needs confirmation, the chat pane shows its preview: a diff for `write_file` / `edit_file`, the command for `bash`, and the content for `read_file`. `Tab` switches between the preview and the conversation; `Ctrl-u` / `Ctrl-d` scroll.

Patterns in `[agent.danger]` (`commands` / `paths`) always require typing `YES` and pressing `Enter`, even under `auto`. This is a guardrail, not a sandbox.

While Dejima is busy, the input line shows `Thinking...` and rejects input. Two `Esc` presses within one second cancel: running commands are killed, and each unanswered tool call is recorded as `Cancelled by user.` Run `/agent` again to turn it off.

<details>
<summary><strong>Approval policies, limits, and matching rules</strong></summary>

- `ask` (default): confirm each call.
- `auto`: run without asking.
- `deny`: hide the tool from the model.

Useful `[agent]` keys:

| Key | Meaning |
| :--- | :--- |
| `shell` | Shell prefix for bash (default `"sh -c"`, e.g. `"zsh -l -c"`). |
| `bash_timeout` | Bash tool's per-command limit in seconds (1–3600; default 120). |
| `max_rounds` | Tool rounds per message (1–10000). **Unset = unlimited.** |

> **Warning:** An unset `max_rounds` lets a tool loop run until the model answers or you cancel. `0` and values outside 1–10000 prevent startup.

`[agent.danger]` lists `commands` and `paths` that escalate to a typed confirmation even under `auto`: the preview highlights the match, and you must type `YES` and press `Enter` (`Esc` denies). The two kinds match differently: `commands` match whole words/phrases anywhere in the shell command text (including inside quoted code, so `python -c "os.system('rm ...')"` is caught), while `paths` are plain substrings without word boundaries, applied to workspace-relative file paths and shell text.

</details>

### Subagents

A subagent is a named assistant you configure once and the model calls on its own — a reviewer, a code explorer, a web searcher. One TOML file per subagent:

```toml
# ~/.config/dejima/subagents/review.toml
system_prompt = '''You are a strict code reviewer. Inspect the workspace with
read_file/bash (git diff, cargo clippy/test) and report findings with file:line
refs.'''
tools = ["read_file", "bash"]   # optional; only listed tools are offered
model = "deepseek-v3"           # optional; default = the session's model
description = "Code review"     # optional; shown in the roster and /subagent

[hyperparams]                   # optional; requires an explicit model
temperature = "0.2"             # a choice id as a string, as declared in models.toml
```

- A subagent sees only the briefing the main model writes plus its own `system_prompt` — never your full chat history.
- Spawns and the subagent's own tool calls follow the `[agent]` approval policies. A subagent cannot call another subagent.
- Completed runs are archived to `sessions/{id}.subagents.toml`. Definitions load at startup.

<details>
<summary><strong>Subagent reference: fields, delegation, and lifecycle</strong></summary>

| Key | Required | Meaning |
| :--- | :--- | :--- |
| **file name** | yes | The subagent's name (`review.toml` → `review`); no spaces or path separators. |
| `system_prompt` | yes | Its persona and instructions. |
| `tools` | no | Whitelist — only these tools are offered; **omitted = none** (a consultation-only subagent that works with any model). |
| `model` | no | Model for the run; must exist in `models.toml`. Required when `hyperparams` is non-empty. |
| `description` | no | One-line label used in the roster and the `/subagent` popup. |
| `hyperparams` | no | Generation-parameter selections: each entry maps a parameter name declared under `[model.hyperparams.<name>]` of the definition's `model` to one of its values, written as a string (`temperature = "0.2"`). A selection overrides the model's default for that subagent's runs only; unselected parameters keep the model default. |

**What the model sees** (the tool's description lists every definition as `name — description (tools: …)`):

| Sent to the model | Never sent |
| :--- | :--- |
| The name, `description` (one line, clipped at 120 characters, `(no description)` if omitted), and the `tools` list (`no tools` if empty). | `system_prompt`, `model`, `hyperparams`. |

The roster appears only when agent mode is on, the protocol supports tool calls, at least one definition exists, and `[agent] subagent` is not `deny`. The whole roster is capped at about 4 KB (`… (roster truncated)`).

**How a delegation runs**

- The model writes the briefing itself and may delegate whenever a subagent fits — you do not have to name one. The subagent sees only that briefing plus its own `system_prompt`, never your conversation.
- You can also request one directly: `/subagent review check the latest commit` sends a validated directive (the name is checked against the roster, so a typo never reaches the model; agent mode must be on and `[agent] subagent` must not be `deny`); the model then composes the briefing and calls the tool. A name-only `/subagent review` leaves the briefing to the conversation context. Forgetting the name is fine — bare `/subagent` opens the roster and `Enter` completes the name for you.
- Each spawn asks for confirmation (`[agent] subagent = "ask" | "auto" | "deny"`); the subagent's own tool calls are approved separately under the same `[agent]` policies.
- Its `tools` whitelist is enforced, it cannot call another subagent, and `max_rounds` caps its loop with the same value as the main loop.
- Declared `hyperparams` are validated at startup against the definition's model (the parameter must exist and the value must be one of its declared values — a typo aborts the start naming the file and key), and the run archive records them.
- The final report returns as the tool result in the same turn, so the model can act on the findings immediately. A round-limited or failed run still delivers what it produced, above a short outcome line.

**Watching a run** — cyan means "subagent":

| Surface | What it shows |
| :--- | :--- |
| busy line | `⠋ subagent review · round 3 · bash: cargo test` |
| chat pane | The narration under a cyan `⠿` label, with its live thinking in a branded block. |
| tool confirmations | `subagent review · Run bash command?` with a cyan title. |
| live Bash pane | `⠋ subagent review · ⚙ bash …` |

Two `Esc` presses cancel the whole turn including the run.

**Files and lifecycle**

- Definitions load once at startup: restart Dejima after adding or editing one; a malformed file aborts startup naming the file and key.
- Every finished or cancelled run is appended to `sessions/{id}.subagents.toml` (definition snapshot, briefing, tool-call trail, report) as an audit trail. A report deleted from the transcript can be copied back from there by hand; deleting the session removes the archive file with it.

</details>

## Configuration

By default, all files live in `~/.config/dejima/`:

```text
~/.config/dejima/
├── models.toml       # required: the models you can chat with
├── config.toml       # optional: defaults, hooks, editor, agent settings, keybindings
├── dejima.log        # error log (auto-created)
├── prompts/          # one Markdown file per system-prompt template
│   └── rust-expert.md
├── snippets/         # one Markdown file per message snippet
│   └── summarize.md
├── subagents/        # optional: one TOML file per subagent
│   └── review.toml
└── sessions/         # chat history (TOML), managed automatically
```

### Custom locations

Every path is overridable. `--root <DIR>` moves all of them under `<DIR>`; individual flags override one path each.

<details>
<summary><strong>Path flags and examples</strong></summary>

| Flag | Default |
| :--- | :--- |
| `--root <DIR>` | `~/.config/dejima` (the base for the rest) |
| `--config <FILE>` | `<root>/config.toml` |
| `--models <FILE>` | `<root>/models.toml` |
| `--prompts <DIR>` | `<root>/prompts/` |
| `--snippets <DIR>` | `<root>/snippets/` |
| `--subagents <DIR>` | `<root>/subagents/` |
| `--sessions <DIR>` | `<root>/sessions/` |
| `--log <FILE>` | `<root>/dejima.log` |

An explicit flag beats `--root`; the environment variable `DEJIMA_ROOT` sets the root when `--root` is absent; otherwise `~/.config/dejima` is used. The root and the `prompts/`, `snippets/`, `subagents/`, and `sessions/` directories are created on startup. To keep one project's chats local while sharing your models and prompts:

```bash
dejima --sessions ./.dejima/sessions
```

For a fully isolated project (its own config, prompts, snippets, models, and sessions):

```bash
dejima --root ./.dejima
```

</details>

### models.toml (required)

| Field | Description |
| :--- | :--- |
| `name` | **Required.** Unique registry key and the model ID sent to the API. |
| `endpoint` | **Required.** URL to POST to (meaning depends on `protocol`). |
| `friendly_name` | Optional. Human label for the `/model` popup and chat headers. |
| `api_key_env` | Optional. Env var holding the API key (never stored on disk). |
| `protocol` | Optional. `"openai"` (default), `"gemini"`, or `"opencode-go"`. |
| `reasoning_replay` | Optional. How stored reasoning is replayed: `"off"` (default), `"present"`, `"strict"`, or `"content"`. See [Reasoning replay](#reasoning-replay). |
| `reasoning_replay_fallback` | Optional. Text used by `"strict"` for turns without stored reasoning (default: empty). Use a single space if your provider rejects an empty value. |
| `hyperparams` | Optional. `[model.hyperparams.<name>]` tables for selectable request parameters. See [Hyperparameters](#hyperparameters). |

A minimal configuration:

```toml
[[model]]
name     = "qwen-32b"
endpoint = "http://localhost:11434/v1/chat/completions"

# For a hosted endpoint, also set the environment variable that holds the key:
# api_key_env = "OPENAI_API_KEY"
```

<details>
<summary><strong>Full multi-provider example and field notes</strong></summary>

```toml
[[model]]
name          = "qwen-32b"
endpoint      = "http://localhost:11434/v1/chat/completions"
friendly_name = "Qwen 32B"
api_key_env   = "OPENAI_API_KEY"
protocol      = "openai"

# Each hyperparameter table is one parameter. `values` are the selectable
# choices; `default` is sent when the session has no explicit selection;
# `path` optionally places the value inside the request body (omitted here, so
# the table key is the root body key).
[model.hyperparams.reasoning_effort]
label = "Reasoning"                 # optional UI label; defaults to the key
values = ["none", "low", "medium", "high"]
default = "medium"

[model.hyperparams.temperature]
values = [0.2, 0.7]
default = 0.7

[[model]]
name          = "gemini-3.8-flash"
endpoint      = "https://generativelanguage.googleapis.com/v1beta"
friendly_name = "Gemini 3.8 Flash"
api_key_env   = "GEMINI_API_KEY"       # export GEMINI_API_KEY=...
protocol      = "gemini"

# Gemini nests settings under generationConfig; a payload supplies the value
# assigned at that root key for each choice.
[model.hyperparams.generationConfig]
label = "Reasoning"
values = ["low", "high"]

[model.hyperparams.generationConfig.payload]
low  = { thinkingConfig = { thinkingLevel = "low" } }
high = { thinkingConfig = { thinkingLevel = "high" } }

# OpenCode Go (OpenAI-compatible Chat Completions). Dejima supplies the
# required stable per-conversation session header automatically.
[[model]]
name          = "deepseek-v4.1-flash"
endpoint      = "https://opencode.ai/inference/go/openai/v1/chat/completions"
friendly_name = "DeepSeek V4.1 Flash"
api_key_env   = "OPENCODE_GO_API_KEY"
protocol      = "opencode-go"

# A local server with arbitrary template arguments (e.g. llama.cpp).
# `path` puts independent selections inside one shared body object; omit it
# to use the table key as a root body key.
[[model]]
name      = "qwen3-local"
endpoint  = "http://localhost:8080/v1/chat/completions"

[model.hyperparams.enable_thinking]
label = "Thinking"
path = ["chat_template_kwargs", "enable_thinking"]
values = [false, true]

[model.hyperparams.target_lang]
label = "Target language"
path = ["chat_template_kwargs", "target_lang"]
values = ["ja", "en"]

[model.hyperparams.glossary]
label = "Glossary"
path = ["chat_template_kwargs", "glossary"]
values = ["none", "project"]

[model.hyperparams.glossary.payload]
none = []
project = [
  { source = "管理画面", target = "dashboard" },
  { source = "プロジェクト", target = "project" },
]
```

> **Breaking change (v4):** the old `reasoning` and `[model.reasoning_payload]` fields are removed. Declare reasoning as `[model.hyperparams.reasoning_effort]` (OpenAI-compatible) or as a `[model.hyperparams.generationConfig]` payload (Gemini). Old configs fail at startup with a migration pointer. Old sessions load, but their stored reasoning selection is not migrated.
>
> Without `path`, the table key is a **literal request-body root key**. `path` is an array of literal JSON keys for nested settings; dots inside a segment are not separators. TOML table keys containing dots must be quoted (for example `[model.hyperparams."my.param"]`). `payload` keys must match the chosen value's canonical text (for example `1.0` becomes `"1"`), so quote numeric or date keys.

</details>

### Reasoning replay

`reasoning_replay` controls what Dejima does with reasoning the model returned in earlier turns. The current turn's reasoning is never replayed to itself; it is stored and used on the next request. "Has reasoning" means the stored message already contains non-empty reasoning from the server's earlier response.

<details>
<summary><strong>Mode reference and provider guidance</strong></summary>

| Mode | When it applies | Messages with reasoning | Messages without reasoning |
| :--- | :--- | :--- | :--- |
| `"off"` | Always | Nothing | Nothing |
| `"present"` | Always (agent on or off) | Replay the reasoning | Nothing |
| `"strict"` | Only while agent mode is on (the request carries tools) | Replay the reasoning | Fallback (empty by default) |
| `"content"` | Always (agent on or off) | Put the reasoning into the message text | Nothing |

- `"present"` replays only what the model actually reasoned about. It also works in plain chat.
- `"strict"` is for providers that require the reasoning field on every previous assistant message once tools are present. Without agent mode it does nothing. `reasoning_replay_fallback` is a placeholder that satisfies that requirement; it is not new reasoning. Use `" "` if your provider rejects an empty value.
- `"content"` is for servers that do not accept a separate reasoning field. It puts the reasoning into the message text and does not satisfy providers that require the separate field (such as DeepSeek).

Provider guidance:

| Model family | Recommended `reasoning_replay` |
| :--- | :--- |
| DeepSeek (agent mode) | `"strict"` — `"present"` can fail; `"content"` does not satisfy the requirement |
| GLM | `"present"`; use `"strict"` if turns without reasoning fail |
| Native Gemini | `"off"` or `"content"`; `"present"`/`"strict"` have no effect |
| Other OpenAI-compatible | `"off"` unless the server supports reasoning replay |

</details>

### Hyperparameters

Each `[model.hyperparams.<name>]` table is one selectable request parameter. There is no `hyperparams` wrapper in the JSON sent to the server: the selected value is inserted at its declared `path` (or at the table key when `path` is omitted). A parameter with neither a selection nor a `default` is omitted.

<details>
<summary><strong>Examples and path/payload details</strong></summary>

For example, with the local-server model above, selecting `enable_thinking = true`, `target_lang = "en"`, and `glossary = "project"` sends:

```json
{
  "model": "qwen3-local",
  "messages": [{ "role": "user", "content": "こんにちは" }],
  "stream": true,
  "stream_options": { "include_usage": true },
  "chat_template_kwargs": {
    "enable_thinking": true,
    "target_lang": "en",
    "glossary": [
      { "source": "管理画面", "target": "dashboard" },
      { "source": "プロジェクト", "target": "project" }
    ]
  }
}
```

- `path = ["chat_template_kwargs", "enable_thinking"]` creates the nested object; multiple parameters with the same parent merge into one object.
- Root-level parameters (no `path`) are added directly at the root, such as `"temperature": 0.7` on a model that declares it.
- A choice without `payload` is sent as its typed scalar (`true`, `"en"`).
- A choice with `payload` sends the payload value; `glossary = "project"` sends the array above.
- If nothing is selected, the key is omitted entirely — the body is unchanged.
- For Gemini, use a path such as `["generationConfig", "thinkingConfig", "thinkingLevel"]`, or target `["generationConfig"]` with a payload object. The selected value is assigned at that path (other fields omitted):

```json
{
  "generationConfig": {
    "thinkingConfig": { "thinkingLevel": "high" }
  }
}
```

Only the selected keys are added; Dejima does not send the `models.toml` table structure itself.

Clearing an explicit selection returns to the declared default; it does not omit the key. To keep a key out of the body, remove its `default` from `models.toml` (or declare a value the provider treats as off). Defaults live only in `models.toml`, so existing sessions pick them up without migration.

</details>

### config.toml (optional)

All keys are optional; the sample comments show the defaults.

<details>
<summary><strong>config.toml reference and sample</strong></summary>

```toml
default_model  = "qwen-32b"     # defaults to the first model in models.toml
default_prompt = "rust-expert"  # optional system prompt template name

hook_command = "terminal-notifier -title 'Dejima' -message 'done'"  # run after every LLM response
hook_shell   = "zsh -l -c"    # shell invocation prefix for both hooks; default "sh -c"

stream_fps   = 10             # streaming redraw cap (1-120); default 30
scroll_step  = 15             # rows scrolled per Ctrl-u/Ctrl-d press (1-500); default 10

editor_command  = "nvim"      # editor for Dejima's editor flows; overrides $EDITOR
preview_command = "glow -p"   # viewer for /preview only; defaults to the editor chain

[keys]
scroll_up = ["ctrl+u", "pageup"]   # add an alternate key
session_new = "N"                  # single spec replaces the default
quit = ["ctrl+c", "ctrl+q"]

# Every tool defaults to "ask".
[agent]
shell        = "sh -c"      # bash tool's shell prefix (whitespace-split); default "sh -c"
bash_timeout = 300          # bash tool's per-run cap in seconds (1-3600); default 120
# max_rounds = 200          # optional tool-round cap per message (1-10000); unset = unlimited
ask_hook_command = "terminal-notifier -title 'Dejima' -message 'approval needed'"  # run when a tool confirmation is shown
bash       = "ask"          # confirm each shell command before it runs
read_file  = "ask"          # confirm reads too (default)
write_file = "ask"          # show a diff and confirm before writing
edit_file  = "ask"          # show a diff and confirm before editing
# To opt out per tool, use "auto" or "deny", e.g.:
# bash      = "auto"
# read_file = "auto"

# Dangerous patterns: even under "auto", a match requires typing YES + Enter.
# commands match whole words/phrases anywhere in the shell command text
#   (quotes/backslashes ignored, case-insensitive): `rm` matches `rm -rf` and
#   `os.system('rm ...')`, but not `rmtree`.
# paths are plain substrings (no word boundaries) matched against
#   workspace-relative file paths and shell command text: `cat .env` is
#   caught, but `.env` also matches `.envrc`. Use absolute fragments like
#   "/tmp/" to catch external scratch writes (bash is not sandboxed).
[agent.danger]
commands = [
  "rm",
  "git reset",
  "git push",
  "git clean",
  "git rm",
  "sudo",
  "dd",
  "mkfs",
  "format",
]
paths = [".env", ".git", "~", "../"]
```

| Field | Description |
| :--- | :--- |
| `default_model` | Model name (must match `models.toml`). Defaults to the first model if omitted. |
| `default_prompt` | Prompt file name in `prompts/` (without extension). |
| `hook_command` | Shell command to run after every LLM response. |
| `hook_shell` | Shell invocation prefix shared by both hooks (default: `"sh -c"`). |
| `stream_fps` | Streaming redraw cap in fps (1–120, default 30). Lower it to reduce CPU usage. |
| `scroll_step` | Rows scrolled per `Ctrl-u`/`Ctrl-d` press, in both the transcript and the tool-approval pane (1–500, default 10). |
| `editor_command` | Editor used by `/editor`, `/edit`, `Ctrl-o`. Overrides `$EDITOR`; defaults to `$EDITOR`, then `vim`. |
| `preview_command` | Viewer for `/preview` only. Falls back to `editor_command` → `$EDITOR` → `vim`. Prefer a blocking command (e.g. `glow -p`, `code -w`); the temp file is removed on return and has no `.md` extension, so extension-based viewers show plain text. |
| `[agent]` | Per-tool approval for agentic mode: `ask` (default), `auto`, or `deny`. `deny` hides the tool from the model. An invalid policy, or a `shell` value that uses an approval keyword, aborts startup. |
| `[agent] subagent` | Approval for the `subagent` tool (default `ask`): every delegation the model requests is confirmed. |
| `[agent] shell` | Shell prefix for the bash tool (default `sh -c`; e.g. `zsh -l -c`). |
| `[agent] ask_hook_command` | Shell command run when a tool-confirmation prompt appears; shares `hook_shell`. |
| `[agent] bash_timeout` | Per-run limit for the bash tool in seconds (1–3600; default 120). |
| `[agent] max_rounds` | Tool-round cap per message (1–10000; unset = unlimited). |
| `[agent.danger]` | `commands` / `paths` patterns that force typed confirmation (`YES` + Enter) even under `auto`. `commands` match shell commands; `paths` match workspace-relative paths (and shell text, so `cat .env` is caught). Matching tolerates formatting and case. Unknown keys abort startup. |

> **Warning:** The shell must accept the command as its final argument (e.g., `zsh -l -c`). Using a bare `hook_shell = "sh"` or `shell = "sh"` will fail at runtime. Prefixes are split on whitespace; quoting is not supported.

> **Hook isolation:** the post-response and approval hooks run in the background with no terminal input. On Unix they start in their own session at lower priority, so a heavy hook is much less likely to stall the UI (it cannot help if the CPU is already saturated). Each run is limited to a fixed **120 seconds**, independent of `[agent] bash_timeout`; after that the process (its whole group on Unix) is killed and a toast is shown. Interactive hooks are not supported.

</details>

### Custom keybindings

Add a `[keys]` section to `config.toml` (see the sample above). Each action id maps to one key spec or an array of specs; a binding replaces that action's default.

<details>
<summary><strong>Key spec syntax and scoping rules</strong></summary>

Every action id and its default keys are listed in the reference tables under [Keybindings](#keybindings). Popup-specific bindings (e.g. `hyperparam_open`, `hyperparam_clear`) are scoped to their popup context; keys that drive the shared list behaviour (`list_next`, `list_prev`, `filter_toggle`, `list_close`) still win first inside a popup, so overriding a popup-specific key onto one of those keys makes the override unreachable.

Free-text typing is not remappable.

Key specs are lowercase key names joined with `+`: modifiers `ctrl`, `alt`, `shift` (optional, informational), then the key — letters/digits (`n`, `N`), `enter`, `esc`, `tab`, `shift+tab`, `backspace`, `delete`, `space`, `up` / `down` / `left` / `right`, `home`, `end`, `pageup`, `pagedown`, `f1`…`f12`.

Unknown action ids, unknown key names, duplicate specs, and two actions sharing a key in the same key context fail at startup with a precise message.

</details>

### System prompts

Templates are stored as Markdown files in `~/.config/dejima/prompts/`. Dejima only lists and reads them; create, edit, rename, and delete the files outside Dejima. `/prompt` selects a template for the session or clears it.

### Message snippets

Fixed messages are stored as Markdown files in `~/.config/dejima/snippets/`. Run `/snippet`, press `Ctrl-f` to filter (then type to narrow), and press `Enter`; the file's body is sent as a normal user message. Files are managed outside Dejima, and an empty file is never sent.

## Privacy

Dejima collects **no telemetry** and makes **no background network requests of its own**. The only network traffic Dejima initiates is the requests you trigger, and they go only to the model service you configured.

### What leaves your machine

Every message you send includes the context you have built up in that session:

- the conversation so far — your messages and the model's replies;
- the system prompt you selected, if any;
- when agent mode is on, the results of the file reads and shell commands the assistant runs for you, which stay in the conversation and are re-sent on later turns;
- a transcript of the conversation, only when you run `/generate-title`.

If your context must not leave your machine, use a model that runs locally, and think twice before enabling agent mode with a remote service.

### What stays on your machine

Your sessions and error logs are saved on your computer as plain text — they are not encrypted. Sessions hold your full conversations, including the output of files read and shell commands run while agent mode was on; the error log holds error messages. Keep your user account protected accordingly.

User-configured hooks (`hook_command`, `ask_hook_command`) run in the background as your own shell commands. They can make network requests if you write them that way; they are not part of Dejima's telemetry.

## Troubleshooting

| Issue | Resolution |
| :--- | :--- |
| **"Failed to load models.toml"** | Create it: `mkdir -p ~/.config/dejima`, then add `models.toml`. |
| **Missing error logs** | Check `~/.config/dejima/dejima.log`. |
| **HTTP 401 / Auth errors** | Set `api_key_env` in `models.toml` and export the variable. |
| **HTTP 400 `MissingSessionID` (OpenCode Go)** | Set `protocol = "opencode-go"` on the model in `models.toml`. |
| **Wrong `$EDITOR`** | Set your system `$EDITOR` variable; it is run via a shell, so full command lines like `open -W -a TextEdit` work (quit the app with ⌘Q to return). |
| **Garbled screen** | Run the shell's `reset` command. |
| **macOS: "cannot be opened" or killed on launch** | Browser downloads are quarantined; run `xattr -d com.apple.quarantine ./dejima` (or download with `curl`/`wget`). |

## License

Dejima is licensed under the [Apache License 2.0](LICENSE). Third-party license notices are published with each release as `THIRD_PARTY_NOTICES.html`.
