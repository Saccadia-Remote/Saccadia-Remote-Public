# Saccadia Remote User Guide

This guide explains how Saccadia Remote works, how to prepare a computer for access, and how to
start and manage a remote session. It is written for people using the application rather than for
server administrators.

Reviewed for client version **0.4.123** on September 23, 2026. Menu and button names below use
the English interface; the same controls are available in the other supported languages.

The screenshots use fictional device IDs and names. No real user, server identity, password, or
private connection information is included.

## Contents

- [How Saccadia Remote works](#how-saccadia-remote-works)
- [Install a client you trust](#install-a-client-you-trust)
- [Understand the main window](#understand-the-main-window)
- [Prepare a computer for incoming access](#prepare-a-computer-for-incoming-access)
- [Connect to another computer](#connect-to-another-computer)
- [Manage recent connections](#manage-recent-connections)
- [Approve an incoming connection](#approve-an-incoming-connection)
- [Control an active remote session](#control-an-active-remote-session)
- [Choose a screen resolution mode](#choose-a-screen-resolution-mode)
- [Navigate a wide desktop with Fit height](#navigate-a-wide-desktop-with-fit-height)
- [Use chat, recording, clipboard, and files](#use-chat-recording-clipboard-and-files)
- [Use console and MCP access](#use-console-and-mcp-access)
- [Change application settings](#change-application-settings)
- [Understand the update package cache](#update-package-cache)
- [Review diagnostic logs and relay health](#review-diagnostic-logs-and-relay-health)
- [Troubleshoot a connection](#troubleshoot-a-connection)
- [Use Saccadia Remote safely](#use-saccadia-remote-safely)

## How Saccadia Remote works

Saccadia Remote connects two copies of the application:

- the **host** is the computer being viewed or controlled;
- the **viewer** is the computer used to connect to the host;
- the Saccadia infrastructure helps the two devices find each other and carries encrypted
  connection messages and, when needed, encrypted session packets.

In everyday use, the process is simple:

1. The host opens Saccadia Remote and shares its 12-digit device ID with a trusted person.
2. The viewer enters that ID and selects **Connect** or presses **Enter** in the ID field.
3. The devices authenticate with a permanent password, a one-time password, saved authorization,
   or explicit approval on the host.
4. The two devices establish fresh encryption keys for that session.
5. The host sends its screen and optional audio; the viewer sends permitted input, chat, clipboard,
   or file data.

Normal session contents are encrypted on one participating device and decrypted on the other.
Coordinator, Edge, and relay services do not receive the session keys required to read the screen,
audio, input, clipboard, chat, or transferred files.

The network can use several relay paths at the same time. A route that becomes slow or unavailable
does not necessarily end the session because another path can continue carrying encrypted packets.
Relays can see ordinary delivery metadata such as addresses, timing, and traffic size, but not the
encrypted content. Saccadia Remote is a remote-access product, not an anonymity service.

For a more detailed explanation, see [Security and Encryption](SECURITY.md),
[Privacy](PRIVACY.md), and [How the system works](ARCHITECTURE.md).

## Install a client you trust

The only official public Saccadia Remote service is
[SaccadiaRemote.com](https://saccadiaremote.com). Its website provides Windows and Linux client packages
already configured for that service.

1. Open the official website in your browser.
2. Download the client package for your platform: Windows x64/x86 or Linux x64.
3. Read and accept the installer disclaimer.
4. Allow the installation to finish and start Saccadia Remote.

On Windows, run the MSI and install the matching .NET 8 Desktop Runtime if prompted.
On Linux, extract the tar.gz package, open a terminal in the extracted directory, and run:

```bash
sudo bash ./install-linux-client.sh
```

Read the terminal notice and type `I ACCEPT` to proceed. The package also includes
`uninstall-linux-client.sh` for removal; run it from a terminal with administrator privileges.
Linux requires .NET 8, not the Windows Desktop Runtime. Follow the installer's dependency link
if the required runtime is missing.

A self-hosting administrator can provide a client configured for a different server. In that case,
confirm the administrator and download source before installing it. A client from another server is
not an official-service client, even if the application looks the same.

Do not disable Windows Defender Firewall to make the application work. The installer registers the
required application and service rules. If a connection is blocked, check the firewall rules or ask
the server administrator for help instead of turning off system-wide protection.

## Understand the main window

![Saccadia Remote main window with fictional IDs](assets/user-guide/main-window.png)

The top part of the window contains the controls needed for most connections:

1. **Your ID** is the 12-digit address of this computer. Use the copy button to share it without
   retyping it.
2. **Set password** creates or changes the permanent password used to authenticate incoming
   connections.
3. **Incoming connection defaults** determine what a newly connected viewer may do. These defaults
   can be changed for each active incoming connection.
4. **Remote Device ID** is where you enter the host ID you want to reach.
5. **Connect**, or **Enter** in the ID field, starts a connection attempt. **Cancel** stops an
   attempt that is still waiting.
6. **Recent connections** keeps useful local history. A tile or compact row can show availability,
   saved authorization, a favorite marker, a custom name, and the last connection time. The list
   button in the tab header switches between tiles and rows.
7. The bottom status bar shows whether the client is connected to signalling, whether its relay is
   available, and which Edge currently carries its signalling connection.

The gear button opens settings. The question-mark button opens the public project documentation in
your browser. When diagnostic logging is enabled, a log-list button appears below Help and opens the
local logs and relay-health window. Donation links are optional and do not affect application
features.

Closing the main window normally returns Saccadia Remote to the Windows notification area instead
of ending it. The tray menu provides **Open**, **Restart**, and **Exit**. **Restart** may request
Windows administrator confirmation because it restarts the Saccadia host and relay services before
starting a fresh GUI, normally in the tray on Windows. Use **Open** to display its main window.
This is different from the GUI-only restart offered after changing the language, which opens the
main window automatically. If confirmation is cancelled or a service cannot restart, the
original client remains open and reports the problem.

Availability is refreshed periodically. A recently disconnected computer can take a short time to
change from online to unavailable, and a computer that has just opened the application may need a
moment to become reachable.

## Prepare a computer for incoming access

Leave Saccadia Remote running on the computer you want to reach. The status bar should indicate that
signalling is connected and that an Edge is assigned.

### Choose an authorization method

**Permanent password**

Select **Set password**, enter a password, and confirm it. The password is kept on the participating
devices and is not sent to the signalling server. The devices use it to prove that they know the
same secret and to create fresh session keys.

**One-time password**

Enable **Allow login with a one-time password** in Settings. The main window then displays a short
temporary password that can be copied or refreshed. It is replaced after a successful use. Share it
only with the person who should connect now.

**Manual approval**

If no password is used, the host can explicitly accept or reject the request. Session traffic is
still encrypted, but this mode relies more strongly on the signalling service to introduce the
correct devices. Use a permanent or one-time password when stronger authentication is required.

### Choose default permissions

The six buttons under **Incoming connection defaults** control:

1. remote keyboard and mouse control, text clipboard, and file operations;
2. host audio streaming;
3. visibility of the physical host cursor;
4. viewer chat messages;
5. viewer-requested screen recording;
6. access to protected Saccadia controls on the host.

Blue means allowed; gray means not allowed. Grant only what the viewer needs. Protected Saccadia
controls are denied by default so a remote user cannot silently change access settings, reveal a
password, or activate sensitive application actions.

## Connect to another computer

1. Ask the host for its 12-digit **Your ID**.
2. Enter the digits in **Remote Device ID**. Spaces are added automatically.
3. Select **Connect**, or press **Enter** while the ID field has focus.
4. Enter the permanent or one-time password when requested, or wait for the host to approve the
   request.
5. If offered and appropriate for that computer, choose whether to remember authorization for a
   future automatic connection.

Saved authorization is not based only on the public device ID. The two devices retain a separate
protected secret and prove it again on later connections. You can remove saved authorization from
the key icon on a recent-connection tile.

Selecting a recent-connection tile fills its ID and starts the same normal connection process. Use
the star to keep an important computer easy to find and the pencil to give it a local friendly name.

Pressing **Enter** uses the same ID validation, version checks, authorization, and connection flow
as the **Connect** button. It does not bypass the host's permissions or approval.

If a previous session to that ID has ended but still appears active, select **Connect** again or
press **Enter** in the ID field. The client clears its previous outgoing-session state and starts
a fresh connection attempt without requiring a GUI restart. This is a reconnect action: using it
for a working session requests that session's disconnection, rather than simply showing its window.
If a connection attempt is still pending, use **Cancel** before trying again.

## Manage recent connections

Recent connections are stored locally for convenience. The application retains up to 30 entries,
placing favorites first and then the most recently used computers.

Tiles are the default. Select the list button in the **Recent connections** tab header to switch to
compact rows; select it again to return to tiles. This choice is remembered after the application
restarts.

![Recent connections displayed as compact rows](assets/user-guide/recent-connections-list.png)

Each tile and row provides the same information and actions:

- the star marks or unmarks a favorite;
- the colored status dot shows the last known availability;
- selecting the ID, name, time, or another free area starts a connection;
- the key shows that saved authorization exists and lets you remove it;
- the pencil changes the local friendly name;
- the reset symbol restores the computer name reported by the remote device;
- the remove button deletes the entry from local history.

Selecting an action button performs only that action; it does not start a connection. Removing a
history entry does not uninstall Saccadia Remote, change the remote computer, or revoke saved
authorization; use the key action separately when you also want to remove that credential.

## Approve an incoming connection

When another user requests manual access, the host sees the viewer ID and can select **Accept** or
**Reject**. Verify the ID through a separate trusted channel before accepting an unexpected request.

After acceptance, the connection appears on the **Incoming connections** tab:

![Incoming connection permissions and chat](assets/user-guide/main-window-incoming.png)

Each incoming card shows the viewer name, ID, and relay-path count. The six permission buttons are
the same controls described above, but changes here affect this active connection instead of future
defaults. Permissions can be enabled or withdrawn while the session is running.

Select an incoming card to open its host-side chat. Select **Disconnect** to end that viewer's
session. Removing saved authorization prevents that viewer from using the previous saved credential
again, but it does not replace the host's permanent password.

## Control an active remote session

![Active Saccadia Remote viewer session](assets/user-guide/remote-session.png)

The session header shows:

- the remote computer name;
- a green **Connected** indicator;
- the remote device ID;
- the number of active and available relay paths.

The large center area is the remote screen. Keyboard and mouse input are sent only when the host has
allowed remote control. The pointer controlled by this viewer follows the current cursor shape
reported by the host even when the separate **Show host cursor** permission is disabled.

**Show host cursor** controls an additional overlay for movement that did not come from the current
viewer. Physical mouse movement on the host appears without a label. Movement from another
Saccadia Remote session is labelled with that viewer computer's name, while movement injected by
Parsec or another external application is labelled **External**. Windows does not reliably identify
the process behind arbitrary injected mouse input, so Saccadia Remote does not guess an application
name. The current viewer's own movement is not duplicated as a host-cursor overlay.

### Use the session master button

The red floating button at the edge of the remote screen is the session **master button**. It keeps
the important viewer controls available even in fullscreen mode, except while Game mode hides it:

- **Left-click** it to switch between the maximized fullscreen view and normal windowed mode.
- **Right-click** it to open the session context menu.
- The button is **draggable**: hold the left mouse button and move it to any convenient place where
  it does not cover useful screen content. Its relative position and the fullscreen preference are
  remembered separately for that remote computer.

![Session master button and context menu](assets/user-guide/remote-session-master-menu.png)

The context menu contains:

| Item | What it does |
| --- | --- |
| **Chat** | Opens or closes the encrypted session chat panel. |
| **Screen recording** | Opens the recording panel. This item is shown only when the host grants recording permission. |
| **Diagnostics** | Shows or hides live viewer-side playback and connection diagnostics over the remote screen. |
| **Audio** | Enables or mutes playback of sound received from the host. A check mark means audio is enabled. |
| **Game mode** | Enables Ctrl+Alt+Shift+Z to switch fullscreen/windowed mode and permits relative mouse input when the host cursor is hidden in fullscreen. Off by default; remembered for this connection. |
| **Max bitrate** | Limits the remote video stream from 1 Mbit/s up to the maximum supported by the current configuration. Unavailable values are disabled. |
| **Picture quality** | Uses adaptive quality or a fixed quality from 50% to 100%. Adaptive mode reacts to current transport conditions. |
| **Encoding** | Selects OpenH264 or Hardware H.264. Hardware uses Media Foundation on Windows and VA-API on Linux; initialization failure falls back to OpenH264. Diagnostics show the actual backend. |
| **Screen resolution** | Selects **Host resolution**, **Viewer resolution**, or **Fit height**. The choice is remembered for this remote computer. See the display modes below. |
| **Send Ctrl+Alt+Del** | Requests the secure attention sequence when the host platform and permissions allow it. |
| **Source: real/test** | Switches between the real remote desktop and the built-in test source used for troubleshooting. Select it again to return to the other source. |
| **Disconnect** | Ends the current remote session. |

Higher bitrate and picture quality can improve fine detail but require more network capacity. If the
picture becomes unstable on a slow connection, try adaptive quality or a lower bitrate before
changing unrelated system settings.

### Choose a screen resolution mode

Right-click the red master button, open **Screen resolution**, and select a mode:

| Mode | What you see |
| --- | --- |
| **Host resolution** | The complete host desktop fits inside the available viewing area, preserving its aspect ratio. A wide multi-monitor desktop may appear small. |
| **Viewer resolution** | Requests a host display resolution matching the viewer's monitor, then fits the resulting desktop inside the viewing area. Support depends on the host platform, display configuration, and permissions. |
| **Fit height** | Uses the host desktop resolution and scales its full height to the viewing area's height. A wide desktop is shown as horizontal pages instead of shrinking everything to fit its width. |

The selected mode is saved separately for each remote computer. **Fit height** changes how the
viewer presents the desktop; it does not request a viewer-sized host display. Switch back to
**Host resolution** when you want to see the whole desktop at once.

A **Viewer resolution** request can show **waiting for host capture** or **waiting for host unlock**.
Let capture start or ask the host user to unlock the computer. The menu then reports the result;
an unsupported or denied request does not guarantee that the host display has changed.

### Navigate a wide desktop with Fit height

Select **Screen resolution > Fit height** in the master-button menu. The viewer divides the desktop
into the minimum number of horizontal pages needed to cover its full width. These are viewing
pages, not physical monitor boundaries: a page can contain parts of two host monitors.

The first page begins at the desktop's left edge and the last ends at its right edge. Adjacent pages
can overlap so that no part of the desktop is omitted. Overlap alone never triggers a switch while
the pointer remains within the current page. If the whole desktop fits at the chosen height, there
is only one page.

**Windowed mode, including a maximized window**

- Use the full-height left and right panels with circular arrow icons beside the remote image.
- Clicking a panel immediately shows the previous or next page; there is no dwell delay.
- The left arrow is disabled on the first page and the right arrow on the last page. Both are
  disabled when only one page is needed.
- Moving either the local or the host pointer does not switch pages in windowed mode.

Maximizing the ordinary window is not the same as entering fullscreen. Left-click the red master
button to switch between windowed mode and the application's fullscreen view.

**Fullscreen mode**

- The arrow panels are hidden. With remote control allowed, move the normal visible pointer into
  the outermost **3 physical pixels** at the left or right edge of the remote image.
- Keep it there for **1 continuous second** to switch in that direction. The buffer remains three
  physical pixels wide regardless of the operating system's display scaling.
- Move out of the edge buffer and back into the rest of the image before the second has elapsed
  to cancel the switch. Leaving the video area, entering a viewer control or overlay, switching to
  another window, or losing remote-control permission also cancels a local edge gesture.
- After a switch, move to the desired edge again to continue or return. Navigation works in both
  directions and stops at the desktop's ends; it does not wrap from last to first.
- A visible host/other-viewer cursor reported outside the current page can also move the view after
  one continuous second outside it. Returning inside cancels the countdown; the destination page
  contains the latest reported cursor position. Host-cursor following requires that cursor telemetry
  to be available, and does not override an edge gesture already in progress locally.

Changing the window size, entering or leaving fullscreen, or changing the host desktop layout
recalculates the pages and starts at the leftmost page. Ordinary video-frame size or quality changes
do not reset a manually selected page or repeatedly rescale the viewing layout.

### Game mode and the escape shortcut

Enable **Game mode** below **Audio** in the session context menu. It is not a global Settings
option and does not turn itself on for other connections. In fullscreen, when the host reports
a hidden cursor, the viewer hides its local pointer and master button and sends mouse movements
instead of screen coordinates. Press **Ctrl+Alt+Shift+Z** to return to windowed mode; press it
again to enter fullscreen. In windowed mode the pointer and button remain available.

On Windows touchscreen hosts, cursor/button flicker remains a known issue under investigation.

**Session** and **File Manager** are vertical tabs in windowed mode, not context-menu commands.
File Manager is available only with the host's remote-interaction permission. Fullscreen hides
the tabs and presents the remote session.

## Use chat, recording, clipboard, and files

**Chat** works independently from keyboard and mouse control when the host grants chat permission.
Messages are part of the encrypted session and are not stored as server-side chat history by the
standard service.

![Session chat panel](assets/user-guide/remote-session-chat.png)

**Screen recording** is available only when the host permits it. Use the panel to start, pause,
stop, and save a recording. Treat recordings as sensitive files: the person operating the viewer is
responsible for consent, lawful use, storage, and sharing.

![Session screen-recording panel](assets/user-guide/remote-session-recording.png)

**Text clipboard** can synchronize supported copied text when remote-control/clipboard permission
is enabled. Avoid copying passwords or other secrets while a session is active unless the other
participant is intended to receive them.

**Files** can be transferred through the session **File Manager** when remote interaction/file
permission is enabled. The left panel shows the viewer computer and the right panel shows the host.
Use the path boxes, Up/Refresh buttons, sortable table headers, multi-selection, filters, and the
center copy buttons to move files or folders in either direction. Copy progress shows file and byte
counts and can be cancelled. If an existing destination is reached, Saccadia Remote asks whether to
overwrite, overwrite all, skip, skip all, or cancel.

Windows-to-Windows sessions can also use the native file clipboard and shell drag-and-drop path.
Linux sessions currently use text clipboard synchronization plus the explicit File Manager for file
movement. Transferred content is encrypted in the session, but the resulting local files have the
normal protection of the destination computer.

## Use console and MCP access

The session **Console** tab starts a shell on the remote computer under its signed-in user's
account. The host must grant **Allow remote console** permission for that session. This permission is
off by default and can be changed in the host's default and active-session permissions.
The tab stays available beside Session and File Manager. Type commands in the field below the
output and press Enter. The output fills the available space; horizontal and vertical scrollbars
appear when needed.

The optional local MCP gateway lets an AI tool inspect authorized sessions, get screenshots and
connection metrics, send input, transfer files, and use the console. In Settings, enable **Allow
MCP control**. Enable **Allow MCP connection and disconnection** separately if the
tool needs to start or end sessions. The remote host must also grant **MCP access** for the
session; the Robot icon identifies this permission. MCP access is off by default. Console commands
also require remote console permission. Revoking either permission prevents further requests.

The gateway is a local stdio process and uses the running client of the same signed-in user. It
does not expose a network listener. Configure your MCP tool to run
`C:\Program Files\Saccadia Remote\Client\McpGateway\SaccadiaRemote.McpGateway.exe` on Windows or
`~/SaccadiaRemote/McpGateway/SaccadiaRemote.McpGateway` on Linux.

## Change application settings

![Saccadia Remote settings](assets/user-guide/settings.png)

The Settings window contains:

- the installed client version and update status;
- the signalling server address;
- the SHA-256 public key pin used to verify the configured server;
- an optional fixed relay listener port;
- the maximum total bandwidth this client contributes as a relay;
- one-time-password access;
- Wake-on-LAN participation;
- local diagnostic logging;
- MCP control and a separate switch for MCP session connection/disconnection;
- interface language.

The screenshot shows the official public service address and its public verification pin. A pin is
not a password or private key: the client uses it to recognize the expected server. A provisioned
installer supplies the values needed for its own server. Do not change the server address or pin
unless a trusted server administrator gave you replacement values. A client configured for another
server joins that independent service.

Leave the relay listener port empty for automatic selection unless a network administrator requires
a fixed port. The aggregate relay limit controls forwarded encrypted traffic; it does not set the
quality of only one viewing session.

Wake-on-LAN is optional and publishes the network profile needed to request that this computer be
woken. Leave it disabled if you do not need it. Diagnostic logging is local and disabled by default;
logs can contain connection metadata, so review them before sharing.

### Update notifications

An available update does not open the main window or bring it to the foreground.
If the window is hidden, minimized or inactive, the offer waits until you open or
focus it. An active main window shows the offer immediately unless settings,
diagnostic logs or another update prompt are open. Choosing **Later** or closing
the prompt does not repeat it on every focus event. Installation requires your consent.

### Update package cache

The updater intentionally retains the verified installer selected for the current update so a
failed or cancelled installer launch can be retried without downloading it again. After the package
passes verification, it removes older Windows MSI or Linux tar.gz installers, including duplicate
copies of the same version, for that same platform and architecture. It does not clean the cache
merely because the main window was opened.

The client updater stores these files in:

- Windows: `%LOCALAPPDATA%\SaccadiaRemote\Updates\<target>\`.
- Linux: `$XDG_CACHE_HOME/SaccadiaRemote/Updates/<target>/`, or
  `~/.cache/SaccadiaRemote/Updates/<target>/` when `XDG_CACHE_HOME` is unset.

Installer diagnostic logs, incomplete downloads, newer-version packages, and files for other
targets are not removed by this retention step. Locked or inaccessible old packages may remain;
cleanup is attempted again when an update package is verified for installation. This is an installer
retention policy, not a limit on every file or extracted directory in the cache. Do not delete files
while an update or installer is running.

### Interface language

The client supports English, Russian, German, French, Spanish, Brazilian Portuguese,
Italian, Turkish, Simplified Chinese, and Japanese. On first launch, it automatically
saves the system UI language without asking you to choose. Unsupported system languages
use English text; the saved preference is not replaced when the system language changes.

To change it, select **Language** in Settings and save. A successful language change offers:

- **Restart**: apply the language by restarting only the GUI, then open and activate the
  main window. Host and relay services keep running, but active GUI-owned sessions may
  be interrupted. This action does not request administrator elevation.
- **Later**, or closing the prompt: keep the saved language for next launch without
  restarting or changing the current interface language.

The confirmation uses the current interface language. If saving fails, no restart is
offered. If the replacement GUI cannot be launched, the current GUI stays running and
shows an error. Ordinary startup, autostart and full restart from the tray retain their
existing behavior; the automatic window opening is specific to a confirmed language restart.

## Review diagnostic logs and relay health

The overlay's **Repair** counter reports recovered packets per minute, including ordered-input
recoveries; it is not a direct network-loss percentage. Missing video packets have a 100 ms
reordering grace before the first repair request. Recovery packets do not start new gap detection.

For detailed Windows host/viewer transport traces without the GUI, run from an elevated terminal:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File "C:\Program Files\Saccadia Remote\Client\Set-TransportDiagnostics.ps1" -Action Enable
```

This enables logging at the next process startup; it does not restart anything. Restart the GUI
and HostService when safe (restarting HostService interrupts incoming sessions). Use `-Action Disable`
and restart the processes after collection. Logs are in `%ProgramData%\Saccadia Remote\logs`
and `%LOCALAPPDATA%\Saccadia Remote\logs`. Detailed traces are limited to 100 MiB per trace file;
collect near the first failure, from both participants, with the time of reproduction.

Select **Enable diagnostic logging** in Settings and then select **Save** when you need to
investigate a problem. A log-list button then appears below Help in the main window. Select it to
open the protected **Diagnostic logs** window.

The left side lists Saccadia Remote log files. Select one file to display its readable text preview
on the right. You can select and copy text from the preview. The checkbox beside each file marks it
for a file operation, and the checkbox in the list header selects or clears every listed file.

- **Copy selected** places the checked complete log files on the clipboard so they can be pasted
  into a folder or attached to a support message. If nothing is checked, the button becomes
  **Copy all**.
- **Delete selected** asks for confirmation and then cleans only the checked files. A log currently
  being written may be cleared and kept in place instead of being removed. Other failures are
  reported without stopping cleanup of the remaining files.
- The preview may show only the newest part of a very large log, but Copy exports the complete file.

The relay panel at the bottom reports the local Windows service, control channel, listener, active
relay pairs, UDP registration, warmup results, and the latest acknowledgement or failure. On
Windows it also checks every required product firewall rule and lists a missing or mismatched rule,
including its direction, protocol, profile scope, action, enabled state, and executable path. This
inspection is read-only: opening or refreshing the window never changes Windows Firewall and does
not contact the server for additional diagnostics.

Logs can contain device IDs, connection times, network addresses, and error details. Read them
before sharing, send only the files needed for the investigation, and disable logging again when
you no longer need it.

## Troubleshoot a connection

### The host appears unavailable

- Confirm that Saccadia Remote is open on the host.
- Check that the host status bar shows signalling connected and an assigned Edge.
- Confirm the complete 12-digit ID.
- Wait a few seconds after the host starts, then retry.
- If an old attempt continues waiting, select **Cancel** before starting another one.

### The application cannot reach the server

- Check ordinary Internet access and the system clock.
- Verify that the client came from the intended server.
- Ensure Windows Defender Firewall contains the installer-created Saccadia Remote rules. The
  **Diagnostic logs** window reports exactly which required rule is missing or misconfigured without
  changing it.
- Do not solve the problem by disabling the firewall globally.
- On a managed or filtered network, ask the administrator whether the configured service is allowed.

### A session connects but a feature does not work

- Check the corresponding permission on the host's **Incoming connections** card.
- For control, clipboard, or files, verify the first permission button.
- For sound, chat, cursor display, recording, or protected UI, verify its separate button.
- Open the session menu and check local Audio, bitrate, quality, and resolution choices.

### A relay is unavailable or missing

- Enable diagnostic logging, save Settings, and open the log-list button below Help.
- In the relay panel, check that the Windows service is running, the control channel is available,
  and UDP rendezvous is registered.
- Review the latest failure and warmup counts. The server keeps underfilled open sessions in a
  cancellation-safe retry index and migrates them when a relay reconnects with a new listener
  generation; normally you do not need to close and recreate the session. A path can remain below
  its target briefly while failure cooldown expires and the replacement is confirmed.
- `SaccadiaRemoteRelay` is installed for automatic start and recovery after both crash and non-crash
  failures. If Windows still reports it as stopped, start it from Services or use tray **Restart**,
  then retain the Critical RelayService log entry for investigation instead of disabling security
  software.
- **Restart** in the tray menu restarts the client and its local host/relay services and may restore
  a transient local state, but repeated failures should be investigated rather than hidden by
  disabling the firewall or other system protection.
- Copy only the relevant logs if you need to send the problem to support.

### All relay paths were lost and the session will not reopen

- First restore Internet/signalling connectivity and check the host is reachable. Losing every
  usable relay path can end the session; recovery of an open session is not the same as reopening
  one that has already closed.
- On current clients, local session invalidation clears the outgoing active state even if the
  disconnect message cannot reach the server.
- Enter the same **Remote Device ID** and select **Connect** or press **Enter**. This starts a fresh
  attempt even if an old active entry was left behind. If an attempt is still pending, cancel it
  first. The normal password or approval requirements still apply.
- If the problem persists, update both clients, record the time of the failure, and collect the
  relevant logs from both participants. Repeated reconnects cannot replace unavailable relay paths.

### Fit height will not switch pages, or the picture jumps

- Confirm **Screen resolution > Fit height** is selected. Host and Viewer resolution modes show
  the whole desktop rather than pages.
- In an ordinary or maximized window, use the side arrow panels. Mouse-edge switching is available
  only in the application's fullscreen mode.
- In fullscreen, remote control must be allowed for a local edge gesture. Hold the visible pointer
  in the last three physical pixels of the image for a full second without entering a viewer overlay
  or changing focus. The first page cannot move left, and the last cannot move right.
- If the desktop fits in one page, no page switch is needed. Overlapping content between pages is
  intentional; it prevents gaps across a wide desktop.
- For cyclic scaling or a page that immediately jumps back in an unchanged window, update both
  clients to the current release and retry. A genuine window-size or host-layout change does rebuild
  the pages from the left edge. Record the viewer's window/fullscreen mode and display scaling when
  reporting a persistent problem.

### The picture or sound is unstable

- Try adaptive picture quality or a lower maximum bitrate.
- Prefer a stable wired or strong Wi-Fi connection when possible.
- Check whether the relay count is falling or repeatedly returning to zero.
- Enable diagnostic logging only while reproducing a problem, then review the logs before sharing
  them with support.

## Use Saccadia Remote safely

- Install clients only from the official service or a self-hosting administrator you trust.
- Confirm a device ID before accepting an unexpected connection.
- Use a permanent or one-time password for stronger authentication.
- Grant only the permissions needed for the task.
- Remove saved authorization when a device or user should no longer reconnect.
- Keep Windows, security software, and Saccadia Remote updated.
- Do not expose passwords, one-time passwords, private keys, or unreviewed diagnostic logs.
- Remember that encryption cannot protect a device already controlled by malware or an untrusted
  application.

Saccadia Remote is provided as-is and does not replace backups, endpoint security, access control,
or an independent recovery method for important computers. Users are responsible for the systems
they connect to and for actions they perform or authorize. See the full
[Disclaimer](DISCLAIMER.md) and [Frequently Asked Questions](FAQ.md).
