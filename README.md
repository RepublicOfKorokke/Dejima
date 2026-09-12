# Dejima (LLM Chat Client)

https://github.com/user-attachments/assets/062cb66c-c09c-42f0-8aea-4468ec83fd08

## Features

| Feature                | Description                                                                                     |
| :--------------------- | :---------------------------------------------------------------------------------------------- |
| **Sessions**           | Create, rename, delete, switch, and branch with auto-save/auto-naming                           |
| **Rich Markdown**      | Full rendering of code blocks, tables, lists, and quotes                                        |
| **Model registry**     | Multiple endpoints, per-model API keys, and OpenAI/Gemini wire protocols                        |
| **Prompt templates**   | Personal library of system prompts (edited in `$EDITOR`)                                        |
| **Streaming chat**     | Per-response telemetry (tokens/s, time-to-first-token)                                          |
| **Reasoning display**  | Support for models with chain-of-thought (e.g., DeepSeek `reasoning_content`, Gemini `thought`) |
| **History editing**    | Edit chat history directly in `$EDITOR`                                                         |
| **Chat scrollbar**     | Scrollable conversation view using `Ctrl-u` / `Ctrl-d`                                          |
| **File context**       | Attach files (`/add`) with contents re-read on every message                                    |
| **Post-response hook** | Execute any shell command after each reply (e.g., desktop notifications)                        |

## Requirements

| Component      | Requirement                                                          |
| :------------- | :------------------------------------------------------------------- |
| **Terminal**   | Color support (ANSI colors) and ideally TrueColor support            |
| **LLM Server** | Any OpenAI-compatible endpoint OR Google Gemini API (native support) |

## Run

No installation is needed:
use the compiled binary from the [releases](https://github.com/RepublicOfKorokke/Dejima/releases) page and start it in a terminal with an interactive TTY:

```bash
./dejima
```

## Configuration

All files live in `~/.config/dejima/`:

```
~/.config/dejima/
├── models.toml       # required: the models you can chat with
├── config.toml       # optional: defaults, post-response hook, [keys] keybindings
├── dejima.log        # error log (auto-created): transcript of error toasts
├── prompts/          # one Markdown file per system-prompt template
│   └── rust-expert.md    # filename (minus .md) is the template name
└── sessions/         # chat history (TOML), managed automatically
```

### models.toml (required)

| Field           | Description                                                                 |
| :-------------- | :-------------------------------------------------------------------------- |
| `name`          | **Required**: Unique registry key and the model ID sent to the API.         |
| `endpoint`      | **Required**: The URL to POST to (depends on `protocol`).                   |
| `friendly_name` | _Optional_: Human label for the `/model` popup and chat headers.            |
| `api_key_env`   | _Optional_: Name of the env var holding the API key (never stored on disk). |
| `protocol`      | _Optional_: Wire protocol: `"openai"` (default) or `"gemini"`.              |

**Example Configuration:**

```toml
[[model]]
name         = "qwen-32b"             # required: unique registry key AND the model id sent to the API
endpoint     = "http://localhost:11434/v1/chat/completions"  # required (meaning depends on protocol)
# friendly_name = "Qwen 32B"          # optional: human label for the /model popup and chat headers
# api_key_env  = "OPENAI_API_KEY"     # optional: env var holding the API key (key never stored on disk)
# protocol     = "openai"             # optional: "openai" (default) or "gemini"

[[model]]
name         = "gemini-3.8-flash"
endpoint     = "https://generativelanguage.googleapis.com/v1beta"
friendly_name = "Gemini 3.8 Flash"
api_key_env  = "GEMINI_API_KEY"       # export GEMINI_API_KEY=...
protocol     = "gemini"
```

### config.toml (optional)

```toml
default_model  = "qwen-32b"     # defaults to the first model in models.toml
default_prompt = "rust-expert"  # optional system prompt template name
# hook_command = "terminal-notifier -title 'Dejima' -message 'done'"  # run after every LLM response
# hook_shell   = "zsh -l -c"    # shell invocation prefix for the hook; default "sh -c"
# stream_fps   = 10             # streaming redraw cap (1-120); default 30
# editor_command = "nvim"      # editor for Dejima's editor flows; overrides $EDITOR
```

| Field            | Description                                                                                                                              |
| :--------------- | :--------------------------------------------------------------------------------------------------------------------------------------- |
| `default_model`  | Model name (must match `models.toml`). Defaults to the first model if omitted.                                                           |
| `default_prompt` | Prompt file name in `prompts/` (without extension).                                                                                      |
| `hook_command`   | Shell command to run after every LLM response.                                                                                           |
| `hook_shell`     | Shell invocation prefix for the hook (default: `"sh -c"`).                                                                               |
| `stream_fps`     | Streaming redraw cap in fps (1–120, default 30). Lower it to reduce CPU usage.                                                           |
| `editor_command` | Editor used by `/editor`, `/edit`, `/preview`, `Ctrl+o` (ADR-0040 shell syntax). Overrides `$EDITOR`; defaults to `$EDITOR`, then `vim`. |

> **Warning**: The shell must accept the command as its final argument (e.g., `zsh -l -c`). Using a bare `hook_shell = "sh"` will fail.

### Custom keybindings

Add a `[keys]` section to `~/.config/dejima/config.toml`. You can map **action ids** to key specs.

```toml
[keys]
scroll_up = ["ctrl+u", "pageup"]   # add an alternate key
session_new = "N"                  # single spec replaces the default
quit = ["ctrl+c", "ctrl+q"]
```

<details>

<summary> Available action ids </summary>

`quit`, `open_editor`, `submit_message`, `cancel`,
`delete_char`, `scroll_up`, `scroll_down`, `completion_next`,
`completion_prev`, `list_next`, `list_prev`, `filter_toggle`, `list_close`,
`text_submit`, `text_cancel`, `session_new`, `session_rename`,
`session_delete`, `session_branch`, `session_preview`, `session_open`,
`confirm_yes`, `confirm_no`, `prompt_select`, `prompt_delete`, `prompt_new`,
`prompt_edit`, `prompt_rename`, `prompt_clear`, `model_select`,
`file_toggle`, `file_select_all`, `file_add`, `file_parent`, `drop_confirm`.

Free-text typing is not remappable.

</details>

Key specs are lowercase key names joined with `+`: modifiers `ctrl`, `alt`,
`shift` (optional, informational), then the key — letters/digits (`n`, `N`),
`enter`, `esc`, `tab`, `shift+tab`, `backspace`, `delete`, `space`, `up` /
`down` / `left` / `right`, `home`, `end`, `pageup`, `pagedown`, `f1`…`f12`.
Unknown action ids, unknown key names, duplicate specs, and two actions
sharing a key in the same popup fail at startup with a precise message.

### System prompts

Templates are stored as Markdown files in `~/.config/dejima/prompts/`.

### Using Dejima

Type a message and press `Enter`. The reply streams in live. Two `Esc` presses within one second abort the stream.

#### Commands

| Command           | What it does                                                         |
| :---------------- | :------------------------------------------------------------------- |
| `/session`        | Open sessions popup: create, rename, delete, branch, switch, preview |
| `/model`          | Choose the active model                                              |
| `/prompt`         | Prompt templates: select, create, edit, rename, delete, clear        |
| `/add`            | Attach file(s) to the conversation                                   |
| `/drop`           | Remove an attached file                                              |
| `/files`          | Show a summary of attached files                                     |
| `/edit`           | Edit the whole chat history in `$EDITOR`                             |
| `/preview`        | Read the chat history in `$EDITOR` (read-only)                       |
| `/editor [text]`  | Compose the pending message in `$EDITOR`                             |
| `/copy`           | Copy the last reply to the clipboard                                 |
| `/regenerate`     | Discard last reply and re-generate (archived to `{id}.trash.toml`)   |
| `/generate-title` | Name the session from the conversation                               |
| `/quit`           | Quit                                                                 |

#### Keyboard Shortcuts

| Keys                                       | Action                             |
| :----------------------------------------- | :--------------------------------- |
| `Enter`                                    | Send message / run command         |
| `/`                                        | Start a slash command              |
| `Tab` / `Shift-Tab` or `Ctrl-n` / `Ctrl-p` | Cycle through command completion   |
| `Esc`                                      | Cancel completion                  |
| `Esc` `Esc` (within 1 s)                   | Abort streaming                    |
| `Ctrl+o`                                   | Compose current input in `$EDITOR` |
| `Ctrl-u` / `Ctrl-d`                        | Scroll chat up / down (10 rows)    |
| `Ctrl+C`                                   | Quit                               |

## Troubleshooting

| Issue                            | Resolution                                                                                                                                                              |
| :------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **"Failed to load models.toml"** | Create `~/.config/dejima/models.toml`                                                                                                                                   |
| **Missing error logs**           | Check `~/.config/dejima/dejima.log`                                                                                                                                     |
| **HTTP 401 / Auth errors**       | Set `api_key_env` in `models.toml`                                                                                                                                      |
| **Wrong `$EDITOR`**              | Set your system `$EDITOR` environment variable; it is run via a shell, so full command lines like `open -W -a TextEdit` work (return to Dejima by quitting the app, ⌘Q) |
| **Garbled screen**               | Run the shell's `reset` command                                                                                                                                         |
