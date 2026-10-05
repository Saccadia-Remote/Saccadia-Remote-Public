# Saccadia Remote Android User Guide

Use Saccadia Remote on your phone to view and control a Windows or Linux computer.
The Android client is a viewer: it does not accept incoming remote sessions or run
a local relay. You can have one outgoing connection at a time, including a connection
that is still waiting for authorization.

This guide covers Android client version **0.4.159**, updated on October 5, 2026.
It requires Android 8 or later and an ARM64 device.
Button names below use the English interface. The same controls are available in
the other supported languages.

## Contents

- [Install and keep your data](#install-and-keep-your-data)
- [Update from your server](#update-from-your-server)
- [Understand the main screen](#understand-the-main-screen)
- [Connect to a computer](#connect-to-a-computer)
- [Keep a session in the background](#keep-a-session-in-the-background)
- [Manage saved devices](#manage-saved-devices)
- [Control the remote screen](#control-the-remote-screen)
- [Scroll on the remote computer](#scroll-on-the-remote-computer)
- [Use the keyboard and shortcuts](#use-the-keyboard-and-shortcuts)
- [Change session options](#change-session-options)
- [Disconnect](#disconnect)
- [Change application settings](#change-application-settings)
- [Share or delete diagnostic logs](#share-or-delete-diagnostic-logs)
- [Use MCP access](#use-mcp-access)
- [Troubleshoot](#troubleshoot)

## Install and keep your data

Install an APK supplied by an operator you trust. The project's only official public
service is [SaccadiaRemote.com](https://saccadiaremote.com). Independent operators run
their own servers; an APK configured for one of those servers belongs to that
installation. Open that server’s Downloads page, choose **Android ARM64**, and
download the APK. The server embeds its bootstrap address, installation identity,
and certificate PIN in the package before signing it.

If Android asks for permission to install an application from your browser or file
manager, review the source before allowing it. Open Saccadia Remote after installation.
The initial server configuration is applied on first launch; later installations over
the same app preserve your saved settings.

Install compatible newer APKs over the existing application. Do not uninstall it to
perform an ordinary update: uninstalling removes this installation's identity,
settings, device history, and saved authorization. Android accepts updates signed
for the installed application; packages signed by a different server may be incompatible.

To prepare the computer that you want to control, follow
[Prepare a computer for incoming access](USER-GUIDE.md#prepare-a-computer-for-incoming-access)
in the desktop guide.

## Update from your server

The app checks for an update through the connected server’s authenticated update
endpoint. When a newer version is available, it offers **Update** or **Later** on the
main screen. The offer waits while you use settings, logs, or a session, and does not
bring a background application to the foreground.

To check manually, open **Settings → Application updates → Check for updates**.
The section shows the current version and the result. Select **Update** to download
a newer version. Disconnect an active session before starting installation.

Updates use the same server package catalog and controlled download queue as the
desktop client. The settings screen shows waiting and download progress, and provides
**Cancel update**. Downloaded data is retained for resumption after a connection loss.
Before opening the installer, the app verifies the complete file’s size and SHA-256,
package identity, version, and signing certificate.

Android may first ask you to allow Saccadia Remote to install applications. Grant
that permission in the system settings if you trust the source, then return to the
app. Confirm the update in Android’s package installer. Installation requires that
system confirmation; the app does not install updates silently. An accepted update
keeps the app’s ID, settings, saved devices, and authorization.

One current verified APK is retained so an interrupted installation can be retried
without another full download. Older update packages for the same target are removed
after verification of the current one.

## Understand the main screen

The main screen contains the server status, your own ID, the connection form, and
your saved devices below the form. The Settings button is in the top bar. A Logs
button also appears there when diagnostic logging is enabled. **Help**, immediately
to the left of Settings, opens this mobile guide in your browser.

The server indicator follows the desktop colors: green means connected, blue means
connecting, and gray means disconnected or reconnecting. Read the adjacent status
text if a connection fails. A connection can be started after the server is connected.

Your phone has its own device ID even though it cannot host sessions. Share that ID
with the computer's owner so they can recognize your incoming request. The button
next to the ID copies it; the Share button opens Android's sharing menu with the ID
as text. The phone is identified to the server by its system device name, with its
model used as a fallback.

## Connect to a computer

1. Make sure the phone and the computer use the same Saccadia server.
2. Ask the computer's owner for the computer's 12-digit ID.
3. Enter it in **Device ID** and select **Connect**. You can also tap the name of a
   saved device to reconnect.
4. Enter the password if requested, or ask the owner to approve the request on the
   computer. The owner can compare your phone's ID with the one you shared.
5. Wait for the remote image to appear.

If you choose to remember authorization during authentication, a later connection
can use it while the host still accepts that authorization. The host's permissions
continue to apply: viewing the screen does not automatically grant keyboard, mouse,
clipboard, audio, private-mode, or MCP access.

Use **Cancel** on the main screen to stop a pending connection. While a session is
connected, **Open session** returns to its image. Close the existing connection before
starting another one.

## Keep a session in the background

Press Home or switch to another application without disconnecting. Saccadia keeps
the outgoing session, including a request waiting for host authorization, in an
Android foreground service. Allow notifications when Android asks so the session
notification can show **Return to session** and **Disconnect**. Notification denial
does not itself end the session.

Return through the notification or the app icon to continue the same connection.
MCP continues to work in the background, including current screenshots of the remote
computer at its original resolution, terminal operations and file transfers. The
video source belongs to the session and continues receiving frames while its display
is hidden. Returning restores the image without a new authorization request.
A hidden session does not keep the phone screen on.
Disconnecting stops the service and removes its notification. Force-stopping the
app or terminating its process ends the connection; it is not automatically recreated.

## Manage saved devices

Each row shows an alias or the computer's name, its ID, and the date and time of the
last connection. The status dot is green for an online device and gray for an offline
device. The star is to the left of that dot; a yellow star marks a favorite.

Tap the star to change favorite status. Favorites appear first. Within both groups,
the most recently connected devices appear before older ones.

Open the menu at the right of a row to manage it:

- **Set alias** assigns a name that is stored on your phone.
- **Reset alias** restores the name reported by the computer.
- **Delete saved authorization** removes the saved outgoing authorization while
  keeping the device record. You may need a password or host approval next time.
- **Delete record** removes the device from the saved-device list.

A red key before the menu means that saved authorization exists. It is an indicator;
use the row menu to delete authorization.

## Control the remote screen

The host sends the image at its own screen resolution. When a session opens, the
phone fits the whole image into the available viewing area while keeping its
proportions. Zooming changes the local view and does not change the host's resolution.

| Gesture on the remote image | Result |
| --- | --- |
| Tap once | Left mouse click at that point |
| Tap twice quickly | Double click at the same point |
| Press and move one finger | Drag with the left mouse button held until you release |
| Press and hold without moving | Right mouse click |
| Spread or pinch two fingers | Zoom the local image |
| Move two fingers together | Move the visible area of the image |

Two-finger gestures only adjust your view; they do not drag objects on the computer.
You can zoom and move the view even when the remote picture is not changing.
Adding a second finger, cancelling a gesture, rotating the phone, or putting the app
in the background releases a held mouse button.

## Scroll on the remote computer

The control strip is below the image in portrait orientation and to its right in
landscape orientation. Its scrolling band controls the remote mouse wheel; it does
not move the local magnified image.

- In portrait, the down arrow is on the left and the up arrow is on the right.
  Swipe along the band to the left to scroll down and to the right to scroll up.
- In landscape, the up arrow is above the band and the down arrow is below it.
  Swipe up or down along the band to scroll in that direction.

An arrow scrolls once immediately. Holding it for two seconds starts repeated
scrolling every 0.2 seconds; releasing it stops the repeat. First tap or move the
pointer into the remote window that should receive scrolling.

## Use the keyboard and shortcuts

The keyboard button is opposite the session-menu button on the control strip:
on the right in portrait, and at the bottom in landscape. Tap it to open Android's
on-screen keyboard. Text entry uses the encrypted clipboard channel and paste, so
the host must allow clipboard synchronization. Use Android's Back control to hide
the keyboard when finished.

The shortcut strip is above the image in portrait and to its left in landscape.
It provides **Esc**, **Enter**, **Alt+Tab**, **Ctrl+C**, **Ctrl+V**, and **Ctrl+Z**.
These act in the focused application on the remote computer. Ctrl+C and Ctrl+V send
the key combination; they do not by themselves copy text between the phone and host.

The button in the corner of this strip opens other keys: **F1–F12**, navigation
keys, and additional combinations. Linux alternatives include **Super+Tab** for
GNOME and **Ctrl+Shift+C / Ctrl+Shift+V** for terminal copy and paste. The exact
meaning depends on the remote desktop and application. In a Linux terminal,
Ctrl+Z normally suspends a process rather than undoing an edit.

## Change session options

Open the session menu from the control strip. It includes diagnostics, sound,
private mode, maximum bitrate, picture quality, encoding, Ctrl+Alt+Del, and
disconnect. Availability depends on host permissions and capabilities.

- **Diagnostics** shows session status information over the image.
- **Sound** enables or disables playback when the host permits audio.
- **Private mode** requests the host's supported privacy behavior. See
  [Private Mode on the host](USER-GUIDE.md#private-mode-on-the-host).
- **Maximum bitrate** limits the requested video traffic rate.
- **Picture quality** selects a quality value from 50% to 100%. **Adaptive** is a
  separate option: changing one does not reset the other. These are video-quality
  controls; local zoom and host screen resolution remain separate.
- **Encoding** selects hardware H.264 or OpenH264 on a supporting host.
- **Ctrl+Alt+Del** requests secure attention on a supporting host with the necessary
  remote-control permission.

The Android menu does not include chat, screen recording, game mode, resolution,
or source selection. The keyboard is available through its separate strip button.

## Disconnect

Choose **Disconnect** in the session menu to end the connection and return to the
main screen. The application stays open. While the session image is visible, the
app keeps the phone's screen awake. After the session closes, the phone's normal
screen timeout applies again.

## Change application settings

Open **Settings** on the main screen, change the values you need, and select **Save**.

- **Server URL** selects the server used for connections. Changing it reconnects
  signalling. Devices on independent servers do not share a device directory.
- **SHA-256 PIN** is available in the server-key section for a server using a
  self-signed certificate. Obtain the correct public-key PIN from its administrator.
- **Language** offers the system language or English, Russian, German, French,
  Spanish, Brazilian Portuguese, Italian, Turkish, Simplified Chinese, and Japanese.
- **Diagnostic logs** enables writing application logs and adds the Logs button to
  the main screen.
- **Application updates** provides manual checking, download progress, installation,
  and cancellation. See [Update from your server](#update-from-your-server).
- **MCP access** allows an authorized local MCP client to control the session.
- **MCP connection** additionally allows that client to connect and disconnect.
  This switch is available only when MCP access is enabled.

There are no local-host password, incoming-session, or relay-service settings on
Android.

## Share or delete diagnostic logs

Enable diagnostic logging in Settings and save, then open **Logs** from the main
screen. The screen lists files with their metadata and selection checkboxes. It
does not display file contents. The action buttons are above the list.

Select the files you need, or use **Select all**:

- **Share ZIP** creates one archive containing only the selected log snapshots and
  opens Android's sharing menu.
- **Delete selected** removes the selected files after confirmation.
- **Delete all** removes all listed files after confirmation.

Enable logging before reproducing a problem. Share the resulting archive only with
someone you trust; diagnostic information can include connection metadata. Logging
may continue creating new files after you delete old ones while it remains enabled.

## Use MCP access

MCP access and MCP connection management are disabled unless you enable them in
Settings. To obtain connection configuration, use **Settings → More → MCP settings**.
This uses Android's sharing menu. Keep the installation token private.

The Android bridge listens only on the phone's loopback interface. An MCP client
running on a computer needs an authorized USB-debugging connection and port
forwarding to reach it. Configure the existing Saccadia MCP gateway with the port
and token from your phone:

```powershell
adb -d forward tcp:62148 tcp:62148
$env:SACCADIA_MCP_ANDROID_PORT = '62148'
$env:SACCADIA_MCP_ANDROID_TOKEN = '<token-from-your-phone>'
```

Start the gateway with its normal stdio configuration after setting these values.
See the [MCP guide](MCP.md) for gateway setup and host permissions. Android's
single-connection limit applies to MCP too; host MCP and remote-control permissions
still apply to individual actions.

## Troubleshoot

| Problem | What to check |
| --- | --- |
| Connect is unavailable | Wait for a green server indicator; check the network, server URL, and certificate PIN. Cancel or close an existing connection. |
| The host cannot be found | Check its ID, server selection, and whether the host application is online. |
| Authorization fails | Ask the host owner to review the request or password. Delete stale saved authorization through the device menu if needed. |
| You can see the image but cannot control it | Ask the host owner to enable remote keyboard and mouse control. |
| On-screen keyboard text does not appear | Check host clipboard permission and focus in the remote text field. |
| Scrolling affects the wrong window | Position the remote pointer over the intended window before using the scrolling strip. |
| Logs are empty | Enable logging, save Settings, reproduce the problem, and reopen Logs. |
| Android rejects a replacement APK | Use a compatible newer package signed for the installed application. Do not uninstall it just to bypass an update error. |

For host setup and permission details, use the [desktop user guide](USER-GUIDE.md).
For server installation, use the [administrator guide](ADMINISTRATOR-GUIDE.md).
