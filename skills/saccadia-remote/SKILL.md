---
name: saccadia-remote
description: Connect to and operate remote computers through Saccadia Remote MCP, using desktop screenshots, coordinate-based mouse input, keyboard shortcuts, remote terminals, and file transfers. Use for remote desktop tasks, including authorized software installation or removal through a graphical interface.
---

# Saccadia Remote

Saccadia MCP can operate a remote desktop, not just report connection metrics.
You can establish a connection yourself, inspect the screen, click visible buttons,
use application menus and dialogs, and install or uninstall software when the user
requests that task. A separate browser or computer-use connector is not needed
to operate the remote screen. Use the actual tools exposed by this MCP server;
client-specific tool names may have a namespace prefix.

## Find or establish the intended session

- Use `list_sessions` to find an existing outgoing session by its name/device ID.
  Reuse it when it is the computer the user intends to operate.
- If the user supplies a computer name instead of its 12-digit device ID, use
  `list_recent_sessions` to resolve it. Ask only if the intended device is ambiguous
  or absent; do not connect to an arbitrary recent device.
- If no session exists, use `connect_session(deviceId)` when connecting to that
  device is within the user's task. You do not need the user to open the viewer
  manually. The local MCP session-management switch must permit this action.
- A `requested` response means an attempt has started, not that it has succeeded.
  Check `get_connection_status(deviceId)` with spaced calls while progress changes,
  then obtain the real `sessionId` from `list_sessions`. Reuse an existing session
  returned by `connect_session` instead of starting duplicate attempts.
- Normal host consent and authentication still apply. If progress requires a
  password or host approval, explain the specific pending step. Stop retrying a
  denied or failed attempt until its cause changes.

An empty session list does not prove that you cannot connect: it can mean no viewer
session is open, or that the host has not granted MCP access yet. Incoming sessions
from `list_incoming_sessions` are viewers accessing this local computer; they are
not outgoing desktops you can operate. Never substitute an incoming session ID.

## Inspect and operate the screen

1. Read `get_session_info(sessionId)` to verify the device and effective rights.
2. Call `get_session_screenshot(sessionId)` and inspect its PNG. The response gives
   its full `width` and `height`, independently of the local viewer window size.
3. Identify the desired visible control. Call `click` with its pixel coordinates,
   `button: "left"`, and `frameWidth`/`frameHeight` equal to that screenshot's
   dimensions. Do not use coordinates from the local laptop screen or viewer frame.
4. Take another screenshot after a meaningful UI change to confirm what happened
   and choose the next action. A successful `sent` response confirms input was sent,
   not that the application completed the requested operation.

For example, if a 1920x1080 screenshot shows the intended button at (1400,900), use:

```json
{"sessionId":"<actual session ID>","button":"left","x":1400,"y":900,"frameWidth":1920,"frameHeight":1080}
```

If an image is displayed at a reduced size, convert the observed position to the
original PNG's pixels. Refresh the screenshot after resolution or monitor changes.
Do not invent coordinates for an unseen dialog.

Use `move_mouse` for hovering, `scroll_mouse` for scrolling (typically +/-120 per
notch), and `mouse_button` for a drag: press, move, release. Release held buttons
when abandoning a drag. Right-click uses `button: "right"`; a double-click uses
two sequential clicks on the same still-visible target.

`send_key_combo` supports shortcuts such as `Alt+Tab`, `Ctrl+S`, `Win+I`, `Tab`,
`Enter`, and `Escape`. It is not an arbitrary Unicode text-entry tool. Check the
published tool schema and key support rather than passing a whole sentence as a key.

## Software installation, removal, and other UI tasks

For a requested installation or removal, you can navigate the remote OS settings,
open an installer, select visible buttons, and complete dialogs using screenshots
and mouse/keyboard tools. Choose the named application and the requested options;
do not modify unrelated software or security settings. Existing user authorization
for that operation applies to the corresponding UI actions.

Observe the actual progress/result screen and verify completion, for example from
the installed application or its entry in the software list. Account for an expected
restart or disconnect if the requested operation replaces Saccadia itself. If an OS
prompt or inaccessible desktop prevents the next step, report that specific obstacle
instead of claiming that MCP cannot operate graphical applications.

## Terminal and files

Use a remote terminal for commands when it is appropriate to the requested task;
use the graphical workflow when the task is about the UI or a visible installer.
`open_terminal` returns a `terminalId`; include it and the correct `sessionId` in
subsequent calls. `write_terminal` executes a command only when its text ends in
an actual newline. `read_terminal` drains buffered output, so preserve output needed
for the task. Wait for the command's result instead of treating input submission as
success. `close_terminal` kills that shell; close shells you created after their work
finishes, while preserving terminals the user wants to keep running.

`upload_files` reads source paths on the local viewer and writes to a directory on
the remote host. `download_files` reads remote source paths and writes to a local
directory. Use `list_remote_files` to inspect remote paths. These tools transfer
files; deletion or software installation uses an authorized terminal or UI workflow.

## Permissions and recovery

The local Saccadia GUI must be running under the gateway's OS account, with MCP
control enabled. Connection/cancellation/disconnection additionally needs the local
session-management setting. The host must grant MCP access for its incoming session;
mouse, keyboard, file, and terminal operations also require remote control permission.
There is no separate console permission switch in the current product.

On permission failure, identify the missing setting and the computer where it must
be enabled. Do not disable protection or change access defaults to bypass a denial.
If a session disappears, refresh `list_sessions`; do not reuse stale IDs or repeatedly
send input to a disconnected session. Use `get_connection_metrics` when diagnosing
connectivity, not as a prerequisite for every desktop action. Disconnect only when
the task calls for it; a user may want an existing viewer session left open.
