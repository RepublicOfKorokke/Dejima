# Dejima (LLM Chat Client)

https://github.com/user-attachments/assets/79bed8be-ec23-44ba-8027-ba5f7b0fcba0

## Features

| Feature                | Description                                                                                         |
| :--------------------- | :-------------------------------------------------------------------------------------------------- |
| **Sessions**           | Create, rename, delete, switch, and branch with auto-save/auto-naming                               |
| **Rich Markdown**      | Full rendering of code blocks, tables, lists, and quotes                                            |
| **Model registry**     | Multiple endpoints, per-model API keys, and OpenAI/Gemini wire protocols                            |
| **Prompt templates**   | Personal library of system prompts (edited in `$EDITOR`)                                            |
| **Streaming chat**     | Per-response telemetry (tokens/s, time-to-first-token)                                              |
| **Reasoning display**  | Support for models with chain-of-thought (e.g., DeepSeek `reasoning_content`, Gemini `thought`)     |
| **History editing**    | Edit chat history directly in `$EDITOR`                                                             |
| **Chat scrollbar**     | Scrollable conversation view using `Ctrl-u` / `Ctrl-d`                                              |
| **Agent mode**         | Let the model use tools (`/agent`): `bash`, file read/write/edit, in a loop until it answers        |
| **Tool approvals**     | Per-tool `ask`/`auto`/`deny` with a diff preview (`Tab` shows the chat); every tool asks by default |
| **Danger guardrails**  | Typed-`YES` confirmation for configured `[agent.danger]` commands and paths                         |
| **Post-response hook** | Execute any shell command after each reply (e.g., desktop notifications)                            |
| **Approval hook**      | Execute any shell command when a tool confirmation appears (`ask_hook_command`)                     |
| **No data collection** | No telemetry, no tracking, and no background work; only your actions trigger activity               |

## Privacy

Dejima collects no telemetry and does nothing in the background. The only
network traffic is the requests you trigger, and they go only to the model
service you have configured — nowhere else.

### What leaves your machine

Every message you send includes the context you have built up in that session:

- the conversation so far — your messages and the model's replies;
- the system prompt you selected, if any;
- when agent mode is on, the results of the file reads and shell commands the
  assistant runs for you, which stay in the conversation and are re-sent on
  later turns;
- a transcript of the conversation, only when you ask for a generated title.

If your context must not leave your machine, use a model that runs locally, and
think twice before enabling agent mode with a remote service.

### What stays on your machine

Your sessions and error logs are saved on your computer as plain text — they are
not encrypted. Sessions hold your full conversations, including the output of
files read and shell commands run while agent mode was on; the error log holds
error messages. Keep your user account protected accordingly.

## Requirements

| Component      | Requirement                                                          |
| :------------- | :------------------------------------------------------------------- |
| **Terminal**   | Color support (ANSI colors) and ideally TrueColor support            |
| **LLM Server** | Any OpenAI-compatible endpoint OR Google Gemini API (native support) |

## Run

No installation is needed. Download the compiled binary from the
[releases](https://github.com/RepublicOfKorokke/Dejima/releases) page and start
it in a terminal with an interactive TTY:

```bash
./dejima
```

> **macOS:** the binary is not notarized, so a browser download (Safari, Chrome)
> gets a quarantine flag and macOS may refuse to launch it. Clear the flag once:
>
> ```bash
> xattr -d com.apple.quarantine ./dejima
> ```
>
> Downloading with `curl` or `wget` avoids the quarantine flag entirely.

## Configuration

By default, all files live in `~/.config/dejima/`:

```
~/.config/dejima/
├── models.toml       # required: the models you can chat with
├── config.toml       # optional: defaults, hooks, editor, agent settings, keybindings
├── dejima.log        # error log (auto-created): transcript of error toasts
├── prompts/          # one Markdown file per system-prompt template
│   └── rust-expert.md    # filename (minus .md) is the template name
└── sessions/         # chat history (TOML), managed automatically
```

### Custom locations

Every path is overridable. `--root <DIR>` moves all of them under `<DIR>`;
individual flags override one path each:

| Flag               | Default                                    |
| :----------------- | :----------------------------------------- |
| `--root <DIR>`     | `~/.config/dejima` (the base for the rest) |
| `--config <FILE>`  | `<root>/config.toml`                       |
| `--models <FILE>`  | `<root>/models.toml`                       |
| `--prompts <DIR>`  | `<root>/prompts/`                          |
| `--sessions <DIR>` | `<root>/sessions/`                         |
| `--log <FILE>`     | `<root>/dejima.log`                        |

An explicit flag beats `--root`; the environment variable `DEJIMA_ROOT` sets the
root when `--root` is absent; otherwise `~/.config/dejima` is used. The root
and the `prompts/`/`sessions/` directories are created on startup. To keep one
project's chats local while sharing your models and prompts:

```bash
dejima --sessions ./.dejima/sessions
```

For a fully isolated project (its own config, prompts, models, and sessions):

```bash
dejima --root ./.dejima
```

### models.toml (required)

| Field               | Description                                                                                                                                            |
| :------------------ | :----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`              | **Required**: Unique registry key and the model ID sent to the API.                                                                                    |
| `endpoint`          | **Required**: The URL to POST to (depends on `protocol`).                                                                                              |
| `friendly_name`     | _Optional_: Human label for the `/model` popup and chat headers.                                                                                       |
| `api_key_env`       | _Optional_: Name of the env var holding the API key (never stored on disk).                                                                            |
| `protocol`          | _Optional_: Wire protocol: `"openai"` (default) or `"gemini"`.                                                                                         |
| `reasoning`         | _Optional_: List of selectable reasoning values (e.g. `["none","low","high"]`); set via `/hyper-params`.                                               |
| `reasoning_payload` | _Optional_ (`[model.reasoning_payload]` sub-table): per-value **non-empty** JSON object merged into the request body (overrides the protocol default). |

**Example Configuration:**

```toml
[[model]]
name         = "qwen-32b"             # required: unique registry key AND the model id sent to the API
endpoint     = "http://localhost:11434/v1/chat/completions"  # required (meaning depends on protocol)
friendly_name = "Qwen 32B"          # optional: human label for the /model popup and chat headers
api_key_env  = "OPENAI_API_KEY"     # optional: env var holding the API key (key never stored on disk)
protocol     = "openai"             # optional: "openai" (default) or "gemini"
reasoning    = ["none", "low", "medium", "high"]   # optional: selectable via /hyper-params

[[model]]
name         = "gemini-3.8-flash"
endpoint     = "https://generativelanguage.googleapis.com/v1beta"
friendly_name = "Gemini 3.8 Flash"
api_key_env  = "GEMINI_API_KEY"       # export GEMINI_API_KEY=...
protocol     = "gemini"
reasoning    = ["low", "high"]        # sent as generationConfig.thinkingConfig.thinkingLevel

# A local server that does not understand `reasoning_effort` (e.g. llama.cpp):
# map each value to the JSON its server expects.
[[model]]
name      = "qwen3-local"
endpoint  = "http://localhost:8080/v1/chat/completions"
reasoning = ["off", "on"]

[model.reasoning_payload]
off = { chat_template_kwargs = { enable_thinking = false } }
on  = { chat_template_kwargs = { enable_thinking = true } }
```

### config.toml (optional)

```toml
default_model  = "qwen-32b"     # defaults to the first model in models.toml
default_prompt = "rust-expert"  # optional system prompt template name
hook_command = "terminal-notifier -title 'Dejima' -message 'done'"  # run after every LLM response
hook_shell   = "zsh -l -c"    # shell invocation prefix for both hooks; default "sh -c"
stream_fps   = 10             # streaming redraw cap (1-120); default 30
scroll_step  = 15             # rows scrolled per Ctrl-u/Ctrl-d press (1-500); default 10
editor_command = "nvim"      # editor for Dejima's editor flows; overrides $EDITOR
preview_command = "glow -p"  # viewer for /preview only; defaults to the editor chain

[keys]
scroll_up = ["ctrl+u", "pageup"]   # add an alternate key
session_new = "N"                  # single spec replaces the default
quit = ["ctrl+c", "ctrl+q"]

# Every tool defaults to "ask".
[agent]
shell      = "sh -c"        # bash tool's shell prefix (whitespace-split); default "sh -c"
bash_timeout = 300          # bash tool's per-run cap in seconds (1-3600); default 120
max_rounds = 200            # optional tool-round cap per message (1-10000); default: unlimited
ask_hook_command = "terminal-notifier -title 'Dejima' -message 'approval needed'"  # run when a tool confirmation is shown
bash       = "ask"          # confirm each shell command before it runs
read_file  = "ask"          # confirm reads too (default)
write_file = "ask"          # show a diff and confirm before writing
edit_file  = "ask"          # show a diff and confirm before editing
# To opt out per tool, use "auto" or "deny", e.g.:
# bash      = "auto"
# read_file = "auto"

# Dangerous patterns: even under "auto", a match requires typing YES + Enter.
# commands match whole words/phrases in shell commands (rm, git reset, ...).
# paths match workspace-relative path fragments (also inside shell commands,
# so `cat .env` is caught). Use absolute fragments like "/tmp/" to catch
# external scratch writes (bash is not sandboxed).
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

| Field                      | Description                                                                                                                                                                                                                                                                           |
| :------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `default_model`            | Model name (must match `models.toml`). Defaults to the first model if omitted.                                                                                                                                                                                                        |
| `default_prompt`           | Prompt file name in `prompts/` (without extension).                                                                                                                                                                                                                                   |
| `hook_command`             | Shell command to run after every LLM response.                                                                                                                                                                                                                                        |
| `hook_shell`               | Shell invocation prefix shared by both hooks (default: `"sh -c"`).                                                                                                                                                                                                                    |
| `stream_fps`               | Streaming redraw cap in fps (1–120, default 30). Lower it to reduce CPU usage.                                                                                                                                                                                                        |
| `scroll_step`              | Rows scrolled per `Ctrl-u`/`Ctrl-d` press, in both the transcript and the tool-approval pane (1–500, default 10).                                                                                                                                                                     |
| `editor_command`           | Editor used by `/editor`, `/edit`, `Ctrl-o`. Overrides `$EDITOR`; defaults to `$EDITOR`, then `vim`.                                                                                                                                                                                  |
| `preview_command`          | Viewer for `/preview` only. Falls back to `editor_command` → `$EDITOR` → `vim`. Prefer a blocking command (e.g. `glow -p`, `code -w`); the temp file is removed on return and has no `.md` extension, so extension-based viewers show plain text.                                     |
| `[agent]`                  | Per-tool approval for agentic mode: `ask` (default), `auto`, or `deny`. `deny` hides the tool from the model. An invalid policy, or a `shell` value that uses an approval keyword, aborts startup.                                                                                    |
| `[agent] shell`            | Shell prefix for the bash tool (default `sh -c`; e.g. `zsh -l -c`).                                                                                                                                                                                                                   |
| `[agent] ask_hook_command` | Shell command run when a tool-confirmation prompt appears; shares `hook_shell`.                                                                                                                                                                                                       |
| `[agent] bash_timeout`     | Per-run limit for the bash tool in seconds (1–3600; default 120).                                                                                                                                                                                                                     |
| `[agent] max_rounds`       | Tool-round cap per message (1–10000; unset = unlimited).                                                                                                                                                                                                                              |
| `[agent.danger]`           | `commands` / `paths` patterns that force typed confirmation (`YES` + Enter) even under `auto`. `commands` match shell commands; `paths` match workspace-relative paths (and shell text, so `cat .env` is caught). Matching tolerates formatting and case. Unknown keys abort startup. |

> **Warning**: The shell must accept the command as its final argument (e.g., `zsh -l -c`). Using a bare `hook_shell = "sh"` or `shell = "sh"` will fail at runtime. Prefixes are split on whitespace; quoting is not supported.

> **Hook isolation**: the post-response and approval hooks run in the background with no terminal input. On Unix they start in their own session at lower priority, so a heavy hook is much less likely to stall the UI (it cannot help if the CPU is already saturated). Each run is limited to a fixed **120 seconds**, independent of `[agent] bash_timeout`; after that the process (its whole group on Unix) is killed and a toast is shown. Interactive hooks are not supported.

### Custom keybindings

Add a `[keys]` section to `config.toml` (see the sample above). Each action id
maps to one key spec or an array of specs; a binding replaces that action's
default.

<details>

<summary> Available action ids </summary>

`quit`, `open_editor`, `submit_message`, `cancel`,
`delete_char`, `scroll_up`, `scroll_down`, `completion_next`,
`completion_prev`, `list_next`, `list_prev`, `filter_toggle`, `list_close`,
`text_submit`, `text_cancel`, `session_new`, `session_rename`,
`session_delete`, `session_branch`, `session_preview`, `session_open`,
`confirm_yes`, `confirm_no`, `toggle_confirm_view`, `prompt_select`,
`prompt_delete`, `prompt_new`,
`prompt_edit`, `prompt_rename`, `prompt_clear`, `model_select`,
`hyperparam_open`, `hyperparam_clear`.

Popup-specific bindings (e.g. `hyperparam_open`, `hyperparam_clear`) are scoped
to their popup context; keys that drive the shared list behaviour
(`list_next`, `list_prev`, `filter_toggle`, `list_close`) still win first inside
a popup, so overriding a popup-specific key onto one of those keys makes the
override unreachable.

Free-text typing is not remappable.

</details>

Key specs are lowercase key names joined with `+`: modifiers `ctrl`, `alt`,
`shift` (optional, informational), then the key — letters/digits (`n`, `N`),
`enter`, `esc`, `tab`, `shift+tab`, `backspace`, `delete`, `space`, `up` /
`down` / `left` / `right`, `home`, `end`, `pageup`, `pagedown`, `f1`…`f12`.
Unknown action ids, unknown key names, duplicate specs, and two actions
sharing a key in the same key context fail at startup with a precise message.

### System prompts

Templates are stored as Markdown files in `~/.config/dejima/prompts/`.

### Using Dejima

Type a message and press `Enter`. The reply streams in live. Two `Esc` presses within one second abort the stream — the same gesture cancels a running agentic tool turn.

#### Commands

| Command           | What it does                                                                        |
| :---------------- | :---------------------------------------------------------------------------------- |
| `/session`        | Open sessions popup: create, rename, delete, branch, switch, preview                |
| `/model`          | Choose the active model                                                             |
| `/hyper-params`   | Set generation hyperparameters (reasoning); two-level value popup, `c` clears       |
| `/prompt`         | Prompt templates: select, create, edit, rename, delete, clear                       |
| `/edit`           | Edit the whole chat history in `$EDITOR`                                            |
| `/preview`        | Read the chat history in `preview_command` (read-only; falls back to the editor)    |
| `/editor [text]`  | Compose the pending message in `$EDITOR`                                            |
| `/copy`           | Copy the last reply to the clipboard                                                |
| `/regenerate`     | Discard last reply and re-generate (archived to `{id}.trash.toml`)                  |
| `/generate-title` | Name the session from the conversation                                              |
| `/agent`          | Toggle agentic mode (tools: bash, file read/write/edit); stays on until toggled off |
| `/quit`           | Quit                                                                                |

#### Agentic Mode

Run `/agent` to let the model call tools in a loop until it replies in plain
text. It works with both OpenAI-compatible and native Gemini models. The tools
are `bash`, `read_file`, `write_file`, and `edit_file`; file tools are confined
to the workspace root.

Approval is per tool in `config.toml` under `[agent]`:

- `ask` (default): confirm each call.
- `auto`: run without asking.
- `deny`: hide the tool from the model.

When a call needs confirmation, the chat pane shows its preview. `Tab` switches
between the preview and the conversation; `Ctrl-u` / `Ctrl-d` scroll.

Useful `[agent]` keys:

| Key            | Meaning                                                         |
| :------------- | :-------------------------------------------------------------- |
| `shell`        | Shell prefix for bash (default `"sh -c"`, e.g. `"zsh -l -c"`).  |
| `bash_timeout` | Bash tool's per-command limit in seconds (1–3600; default 120). |
| `max_rounds`   | Tool rounds per message (1–10000). **Unset = unlimited.**       |

> **Warning:** An unset `max_rounds` lets a tool loop run until the model
> answers or you cancel. `0` and values outside 1–10000 prevent startup.

`[agent.danger]` lists `commands` and `paths` that escalate to a typed
confirmation even under `auto`: the preview highlights the match, and you must
type `YES` and press `Enter` (`Esc` denies). This is a guardrail, not a
sandbox; bash itself is not confined.

While Dejima is busy, the input line shows `Thinking...` and rejects input. Two
`Esc` presses within one second cancel: running commands are killed, and each
unanswered tool call is recorded as `Cancelled by user.`

Run `/agent` again to turn it off.

#### Keyboard Shortcuts

| Keys                                       | Action                                         |
| :----------------------------------------- | :--------------------------------------------- |
| `Enter`                                    | Send message / run command                     |
| `/`                                        | Start a slash command                          |
| `Tab` / `Shift-Tab` or `Ctrl-n` / `Ctrl-p` | Cycle through command completion               |
| `Esc`                                      | Cancel completion                              |
| `Esc` `Esc` (within 1 s)                   | Abort streaming / cancel a tool turn           |
| `Ctrl-o`                                   | Compose current input in `$EDITOR`             |
| `Ctrl-u` / `Ctrl-d`                        | Scroll chat up / down (default 10 rows)        |
| `Tab` (while a tool prompt is pending)     | Switch between the diff preview and the chat   |
| `Ctrl-c`                                   | Clear the input; quit when empty or popup open |

## Troubleshooting

| Issue                                             | Resolution                                                                                                                                                              |
| :------------------------------------------------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **"Failed to load models.toml"**                  | Create `~/.config/dejima/models.toml`                                                                                                                                   |
| **Missing error logs**                            | Check `~/.config/dejima/dejima.log`                                                                                                                                     |
| **HTTP 401 / Auth errors**                        | Set `api_key_env` in `models.toml`                                                                                                                                      |
| **Wrong `$EDITOR`**                               | Set your system `$EDITOR` environment variable; it is run via a shell, so full command lines like `open -W -a TextEdit` work (return to Dejima by quitting the app, ⌘Q) |
| **Garbled screen**                                | Run the shell's `reset` command                                                                                                                                         |
| **macOS: "cannot be opened" or killed on launch** | Browser downloads are quarantined; run `xattr -d com.apple.quarantine ./dejima` (or download with `curl`/`wget`)                                                        |
