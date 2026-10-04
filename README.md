![Codelyxa](Codelyxa-banner.png)

# Codelyxa — VS Code AI Agent

Codelyxa turns supported browser AI chats into an autonomous coding agent that
operates directly on your VS Code workspace. Instead of copy-pasting code, you
chat naturally and the AI reads, writes, edits, deletes files and runs terminal
commands through a local bridge and the Model Context Protocol (MCP).

- Version: 0.1.0
- License: MIT
- Platform: VS Code (Windows / macOS / Linux)

---

## Features

- **Read** files and directories (recursive listing, line ranges)
- **Edit** existing files surgically (replace exact line ranges)
- **Create** new files or overwrite deliberately
- **Delete** explicitly requested files (never broad cleanup)
- **Move, rename, copy** files
- **Run terminal** / shell commands (tests, builds, formatters, git)
- **Memory** folder for durable notes across sessions
- **On/off toggle** in the popup
- **Debug logging** with a verbose terminal mode
- Discovers the exact MCP tool names at runtime

## Supported AI sites

| Site | URL |
|------|-----|
| DeepSeek | chat.deepseek.com |
| ChatGPT | chatgpt.com |
| Gemini | gemini.google.com |
| Kimi | kimi.ai |
| GLM | chat.z.ai |
| Qwen | chat.qwen.ai |
| Arena | arena.ai |
| Meta AI | meta.ai |

## Architecture

```text
ChatGPT / DeepSeek / Gemini / Kimi / GLM / Qwen / Arena / Meta AI
                         |
                 Codelyxa Extension
                         |
                 localhost WebSocket
                         |
                  Codelyxa Bridge
                         |
          workspace_mcp_server.py (stdio MCP)
                         |
                     VS Code
                         |
               Workspace + Terminal
```

The only socket in the whole stack is the bridge WebSocket at
ws://127.0.0.1:17613, used by the browser extension. Everything else
communicates over stdio — no HTTP port is involved.

## Setup

### 1. Install the browser extension

Chrome or Edge:

1. Open chrome://extensions (or edge://extensions)
2. Enable **Developer mode**
3. Select **Load unpacked**
4. Choose the codelyxa-extension directory

### 2. Open your workspace in VS Code

Open the project you want Codelyxa to control. The bridge starts the
workspace MCP server automatically from config.json — no separate setup.

### 3. Start the bridge

Windows:

```text
start.bat
```

macOS / Linux:

```text
./MacOS_Start.command
```

Keep the bridge window open while you use Codelyxa. It prints live status and
logs every tool call (see **Verbose terminal** below).

### 4. Start an AI session

Open one of the supported AI sites and start Codelyxa. The agent sends its
first command automatically, then works turn by turn.

## How to use

Just chat with the AI as you normally would, for example:

```text
Create a file hello.py that prints Hello World
Read main.py and fix the bug on line 12
Run the tests and show me what failed
Refactor utils.js to use async/await
```

The agent sends one command at a time, waits for the result, then continues.
You can watch each step as a chip under the reply, and expand it to see the
arguments and output.

## The 15 tools

```text
READ
  list_files, read_file, list_files_code, read_file_code
EDIT
  write_file, create_file_code, replace_lines, replace_lines_code
FILE
  move_file_code, rename_file_code, copy_file_code
DELETE
  delete_file, delete_file_code
TERMINAL
  run_command, execute_shell_command_code
```

If the connected MCP server does not expose a capability, Codelyxa reports that
instead of pretending the operation succeeded.

## Memory

After finishing a task the agent may save a short note to `Codelyxa/Memory/` and
add a line to the index. At the start of a session it reads that index to
recall prior work, so context survives across sessions.

## Master on/off switch

The extension popup has a toggle. When off, the background worker opens no
socket and makes no reconnect attempts, so the browser logs no connection
errors. The state is saved and restored on startup.

## Verbose terminal

By default the bridge terminal shows status, errors and warnings. To also see
every tool call and server message, set `Clyxa_VERBOSE=1` before running
the bridge (already enabled in start.bat; add `rem` to disable).

Log file: `logs/bridge_debug.log`

## Configuration

config.json declares the MCP servers:

```json
{
  "mcpServers": {
    "vscode": {
      "command": "workspace_mcp_server.py",
      "args": []
    }
  }
}
```

The workspace server speaks stdio directly — no HTTP port is involved.

## Safety

The agent is instructed to:

- read before editing
- make minimal, surgical edits
- use exact discovered tool schemas
- require an explicit user request before deletion
- avoid broad recursive deletion
- avoid interactive terminal commands
- verify meaningful changes with project tooling when appropriate

## Troubleshooting

- **Bridge offline** — run start.bat and keep the window open.
- **Extension shows errors** — start the bridge, or switch the extension
  off in the popup to silence connection attempts.
- **Agent does not act** — make sure you are on a supported site and the
  bridge is connected (green dot in the popup).
- **File edits look wrong** — the agent reads before editing; if a site renders
  markdown oddly, reload the page.

## License

MIT. See the LICENSE file.
