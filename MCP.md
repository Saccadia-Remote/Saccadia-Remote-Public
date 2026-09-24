# MCP integration

Saccadia Remote 0.4.125 includes a local Model Context Protocol (MCP) gateway in
the Windows and Linux **client**. An MCP-capable AI application can use it to work
with remote computers through the Saccadia viewer: inspect sessions, capture the
host desktop, send input, browse and transfer files, and run a remote terminal.

The gateway is an optional local process. The AI application starts it using the
[MCP stdio transport](https://modelcontextprotocol.io/specification/2025-11-25/basic/transports).
It talks to the already-running Saccadia GUI of the **same operating-system user**.
It does not provide a public HTTP endpoint, listen on a network port, or run on the
Coordinator, Edge, or ServerRelay. The normal encrypted Saccadia session carries
commands and results between viewer and host.

## Before you start

1. Install current Saccadia clients on the **viewer** (the computer running the
   AI application) and **host** (the computer being controlled). This guide covers
   the features in 0.4.125; older clients may not understand these permissions.
2. Start and keep the Saccadia GUI open on the viewer under the same OS account
   that runs the AI application. The gateway needs that GUI's live sessions.
3. In viewer **Settings**, enable **Allow MCP control** and save. This switch is
   off by default.
4. If the AI application must connect, cancel attempts, or disconnect sessions,
   also enable **Allow MCP connection and disconnection**. This separate switch
   is off by default. Browsing recent and incoming sessions does not require it.
5. On the host, enable **MCP access** (robot icon) for the intended incoming
   connection. It is off by default both in **incoming connections by default**
   and in the permissions of an active incoming session. The host can grant it
   for one session without enabling it as a general default.
6. For keyboard, mouse, file operations, or terminal access, the host must also
   grant **Allow remote control, clipboard, and files** for that session.
   Terminal access follows this control permission; there is no separate console
   permission switch. The host can revoke permissions during the session.

Normal Saccadia authentication or host approval still applies to a new connection.
Knowing a device ID, starting the MCP gateway, or enabling the local viewer switch
does not authorize access to a host by itself.

### Permission matrix

| Operation | Viewer: Allow MCP control | Viewer: Allow MCP connection and disconnection | Host: MCP access | Host: remote control |
|---|---|---|---|---|
| List recent outgoing connections and incoming sessions | Yes | No | No | No |
| Connect, check/cancel a connection attempt, disconnect an outgoing or incoming session | Yes | Yes | See note below | No |
| List active outgoing sessions, read session info or metrics, capture desktop | Yes | No | Yes | No |
| Keyboard/mouse input, remote files, transfers, remote terminal | Yes | No | Yes | Yes |

For a new connection, normal host authentication and approval happen first. An
outgoing session appears in `list_sessions` only after the host grants MCP access;
`disconnect_session` also requires that access. Disconnecting an **incoming**
session is an action on the local host and needs the viewer-side session-management
switch, but not the remote viewer's MCP permission. A host may restrict or revoke
either host permission at any time; review the effective rights of the active
incoming session, not only its defaults.

## Configure an MCP client

Point the AI application's **local stdio MCP server** configuration at the gateway
executable. The following is a common JSON shape; configuration keys and file
locations vary by MCP application.

Windows example:

```json
{
  "mcpServers": {
    "saccadia": {
      "command": "C:\\Program Files\\Saccadia Remote\\Client\\McpGateway\\SaccadiaRemote.McpGateway.exe"
    }
  }
}
```

Linux example (replace `USER` and use the actual absolute installation path):

```json
{
  "mcpServers": {
    "saccadia": {
      "command": "/home/USER/SaccadiaRemote/McpGateway/SaccadiaRemote.McpGateway"
    }
  }
}
```

The default Linux installation is under `~/SaccadiaRemote`; expand `~` in a
configuration file because some MCP applications do not expand shell shortcuts.
If a custom installation directory was chosen, locate the `McpGateway` subfolder
of that client installation. Configure the AI application on the **viewer**, not on
the remote host or Coordinator. Restart or reload the AI application's MCP servers
after changing its configuration. The Saccadia GUI must remain running.

An MCP client launches the gateway as a subprocess and exchanges protocol
messages on standard input/output. Running the executable in a terminal without
an MCP client will not display an interactive command prompt. To inspect available
tools manually, use an MCP client such as the
[MCP Inspector](https://modelcontextprotocol.io/docs/tools/inspector) and select
**stdio** with this executable as its command.

## Available tools

`sessionId` is a temporary ID for an open viewer window, returned by
`list_sessions`. It is **not** the 12-digit device ID used for `connect_session`.
It becomes invalid when that session closes. `terminalId` identifies one remote
shell within a session and is returned by `open_terminal`.

| Task | MCP tools | Important inputs or behavior |
|---|---|---|
| Discover sessions | `list_sessions`, `list_recent_sessions`, `list_incoming_sessions` | Active outgoing sessions with MCP access; recent outgoing history without saved credentials; active incoming sessions with host-side permissions. |
| Manage connections | `connect_session`, `get_connection_status`, `cancel_connection`, `disconnect_session`, `disconnect_incoming_session` | Connect by 12-digit `deviceId`; disconnect by the appropriate `sessionId`. Requires the separate local management switch. |
| Inspect a host | `get_session_info`, `get_connection_metrics`, `get_session_screenshot` | Client and desktop details, permissions, relay/connection diagnostics, or a PNG desktop image. |
| Control desktop | `move_mouse`, `scroll_mouse`, `mouse_button`, `click`, `send_key_combo` | Mouse coordinates are host desktop pixels; keyboard combinations include forms such as `Ctrl+S`. |
| Work with files | `list_remote_files`, `upload_files`, `download_files` | Browse a remote path; upload local viewer files to a remote directory; download remote files to a local viewer directory. |
| Use a shell | `open_terminal`, `read_terminal`, `write_terminal`, `resize_terminal`, `close_terminal` | A persistent remote shell with its own `terminalId`; columns/rows are character dimensions. |

All 24 tools require **Allow MCP control** on the local viewer. Tools using an
open outgoing session also require the host's MCP permission. The last three rows
need the host's remote-control permission. Session-management tools have the
additional local switch described above.

### Connection and session IDs

`connect_session` starts the ordinary Saccadia connection flow. Its `requested`
status means the attempt started, **not** that the session is already open or
authorized. Use `get_connection_status` with the same `deviceId` and then
`list_sessions` to obtain an active `sessionId`. The host can still require a
password or manual approval, or refuse MCP access. `cancel_connection` applies
to an attempt in progress. `disconnect_incoming_session` uses an ID from
`list_incoming_sessions`; do not substitute an outgoing session ID.

### Screenshots and pointer coordinates

`get_session_screenshot` returns a PNG image and its width and height. It
temporarily asks for a video frame at the **host desktop's resolution**, even
when the viewer window is much smaller, and then restores the viewer's usual
stream size. The PNG is made from the received video frame, so existing video
compression may still affect fine text. A frame must be available; retry after
the session begins streaming if capture times out.

For `move_mouse`, `scroll_mouse`, `mouse_button`, and `click`, supply `x` and `y`
in **host desktop pixels**, together with `frameWidth` and `frameHeight` from
the most recent screenshot. Do not use the AI application's preview size or the
viewer window size. Capture again after a host display-size change. `mouse_button`
has a `pressed` boolean for press/release; `click` performs both actions. For
scrolling, `delta` is typically 120 per wheel notch.

### Files and remote terminal

For `upload_files`, `sourcePaths` refers to paths on the **viewer** and
`destinationDirectory` to the **host**. For `download_files`, `sourcePaths`
refers to paths on the **host** and `destinationDirectory` to the **viewer**.
`list_remote_files` accepts an optional remote `path`. Transfers use Saccadia's
file channel; a tool may report `completed: false` if a transfer was cancelled
or could not finish. Use absolute paths when ambiguity is possible.

`open_terminal` accepts `columns` and `rows` and returns a `terminalId`.
On Windows the host starts PowerShell through ConPTY; on Linux it uses a PTY
with bash or sh. The shell runs as the signed-in host user, **not** as a server
or system account. `write_terminal` sends text; include a newline to execute
a command. `read_terminal` returns and clears the output buffered since the
last read, so poll it for long-running commands. `resize_terminal` changes
character dimensions. `close_terminal` ends the shell process. MCP terminals
are separate from the session's visible Console tab.

## Security and troubleshooting

- Grant host **MCP access** only to viewers you trust with AI-assisted access.
  It is independent of the viewer's local switch. A host can deny it by default
  and grant it only for a specific incoming session.
- The remote-control permission is powerful: with MCP access it also permits
  terminal commands, input, and file operations. Review proposed AI actions,
  especially commands that modify data or copy files.
- The local gateway uses the live GUI of the same OS user. If tools report that
  MCP control is disabled, enable the setting in that running client. If the
  gateway cannot reach the GUI, check the account and that the GUI is open.
- If `list_sessions` is empty while a viewer session is visible, check the host's
  **MCP access** permission for that active session. If input, files, or terminal
  are denied, check **Allow remote control, clipboard, and files** as well.
- If connecting or disconnecting is denied, enable **Allow MCP connection and
  disconnection** on the viewer. Host authentication is still required.
- If a tool says a viewer session is no longer open, call `list_sessions` again
  and use its new `sessionId`. For an incoming session, use
  `list_incoming_sessions` instead.
- Do not expose the gateway through a generic network-to-stdio bridge unless
  you provide your own authentication and access controls. Its intended trust
  boundary is the local, same-user MCP application.

See the [User Guide](USER-GUIDE.md#use-console-and-mcp-access) for the Console
tab and permissions UI, and [Security](SECURITY.md) for the remote session's
encryption and authentication model.
