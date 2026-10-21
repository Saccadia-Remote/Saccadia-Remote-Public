# Server and Edge installation guide

This guide is for administrators who want to run a private Saccadia Remote installation. The
application source code is not included. The packages contain published service binaries,
installation scripts, configuration templates, and third-party notices.

## Downloads

Current package version: **0.4.172**.

| Role | Operating system | Package |
|---|---|---|
| Server full node | Linux x64 | [Download](packages/SaccadiaRemote-Server-0.4.172-linux-x64.tar.gz) |
| Edge only | Linux x64 | [Download](packages/SaccadiaRemote-Edge-0.4.172-linux-x64.tar.gz) |
| ServerRelay only | Linux x64 | [Download](packages/SaccadiaRemote-ServerRelay-0.4.172-linux-x64.tar.gz) |
| Server full node | Windows x64 | [Download](packages/SaccadiaRemote-Server-0.4.172-windows-x64.zip) |
| Edge only | Windows x64 | [Download](packages/SaccadiaRemote-Edge-0.4.172-windows-x64.zip) |
| ServerRelay only | Windows x64 | [Download](packages/SaccadiaRemote-ServerRelay-0.4.172-windows-x64.zip) |

Verify a downloaded archive against [SHA256SUMS](packages/SHA256SUMS) before extracting it.

The optional [MCP gateway](https://github.com/Saccadia-Remote/Saccadia-Remote-Public/blob/master/MCP.md)
belongs to the installed Windows/Linux
**client**, not these Server, Edge, or ServerRelay packages. It runs locally on
the viewer through stdio and requires separate permissions on the viewer and host.
There is no MCP port to expose on Coordinator or Edge.

## Installation consent

Every package includes `DISCLAIMER.md`. Installation is blocked until the administrator explicitly
accepts that the software is supplied **as is**, without a warranty or guaranteed service level,
and that installing it can have unintended consequences. An interactive installation displays the
notice and requires the exact response `I ACCEPT`. It then prints the selected role, endpoints,
ports, certificate sources, persistent-data location, and operational reminders. No system change
is made until the administrator reviews that plan and types `INSTALL`.

For deliberate unattended automation, Linux requires both exact environment values
`SACCADIA_ACCEPT_DISCLAIMER=YES` and `SACCADIA_CONFIRM_INSTALLATION=YES`; Windows requires both
`-AcceptDisclaimer` and `-ConfirmInstallation`. Do not use these options until the notice and the
complete command have been reviewed.

## Network planning

Choose one stable DNS name or public IP for Coordinator and one for every Edge. Configure NAT and
host/network firewalls as applicable.

| Default port | Protocol | Purpose |
|---|---|---|
| 5000 | TCP | Client bootstrap |
| 5000 | UDP | Direct-connect rendezvous |
| 5443 | TCP | Browser management, diagnostics, and client downloads |
| 7000 | TCP | Coordinator-to-Edge mTLS |
| 5100 | TCP | First Edge client WSS; use another port for additional local Edge nodes |
| 5001 | UDP | Standalone server-relay listener, when installed |

Only management 5443 is intended for a browser. Port 5000 is client bootstrap, 7000 is cluster mTLS,
and long-lived client WebSockets terminate directly on Edge.

## Network map

Open the protected `/status` page and press **Map** to show approximate IP-based
locations of users, client relays, server relays, the Coordinator and Edge nodes.
Online nodes are green, inactive nodes grey; marker shapes distinguish node types.
The button is grey and disabled when the local GeoIP database is unavailable.
Nodes without real coordinates are counted separately and are not placed on the map.

The panel captures data only when you press Map. It has no zoom, pan, marker
actions or automatic refresh; press Map again for a new capture. Ordinary status
polling does not change an open map. The map background is bundled locally and
does not contact an external mapping service.

Captures and images are not stored. Only the last real location of each known
node (type, ID, name, city, country and coordinates) is retained in
`data/network-map.db` under the Coordinator's persistent storage directory so
disconnected nodes can appear grey. IP addresses, connection secrets and online
state are not stored in this database. Online state comes from live registries;
synthetic relay-selection coordinates are excluded. The API `/api/status/map`
requires the same administrator session as the detailed status page.

## Protect the detailed status page

The overview page and its aggregate `/api/overview` data remain public. The detailed `/status` page
and `/api/status` require one administrator password; detailed topology, resources, certificates,
and protection state are not part of the public response. Downloads remain public, and `/health`
exposes only minimal liveness for service orchestration. Configure the password before opening the
status page. Linux full-node commands run inside the Coordinator container:

In the browser header, Status is beside the Server brand. After authentication, Sign out appears
beside Status; Overview and Downloads remain in the right-hand public navigation group. Sign out
deletes the management session and returns the browser to public Overview.

```bash
cd /opt/saccadia-remote
sudo docker compose -f deploy/linux/compose.yaml exec server \
  dotnet /app/SaccadiaRemote.ServerService.dll status-password set
sudo docker compose -f deploy/linux/compose.yaml exec server \
  dotnet /app/SaccadiaRemote.ServerService.dll status-password status
sudo docker compose -f deploy/linux/compose.yaml exec server \
  dotnet /app/SaccadiaRemote.ServerService.dll status-password reset
```

Windows full-node commands require an elevated terminal:

```powershell
.\Coordinator\SaccadiaRemote.ServerService.exe status-password set
.\Coordinator\SaccadiaRemote.ServerService.exe status-password status
.\Coordinator\SaccadiaRemote.ServerService.exe status-password reset
```

The `set` input is hidden and confirmed. Automation may pipe a secret only with explicit
`--password-stdin`; `set --generate` prints one strong password once. The `status` subcommand reports
whether a password is configured without exposing it. The `reset` subcommand immediately invalidates
all browser sessions and blocks detailed browser status until a new password is set. The stored file
contains a salted password verifier, never the plaintext password.

Login is limited globally to five attempts in a rolling 60-second interval, regardless of source IP.
If an attacker occupies that window, inspect the same complete status snapshot without HTTP. Use
`status` for a one-time snapshot or `status --watch` for continuously refreshed output:

```bash
sudo docker compose -f deploy/linux/compose.yaml exec server \
  dotnet /app/SaccadiaRemote.ServerService.dll status
sudo docker compose -f deploy/linux/compose.yaml exec server \
  dotnet /app/SaccadiaRemote.ServerService.dll status --watch
```

```powershell
.\Coordinator\SaccadiaRemote.ServerService.exe status
.\Coordinator\SaccadiaRemote.ServerService.exe status --watch
```

The command uses administrator-only local IPC, does not consume browser login attempts, and does not
accept or require the status password. Every browser status field is printed as a grep-friendly text
path. Press Ctrl+C to stop watch mode.

## Linux full node

Requirements: x86-64 Linux, root access, Docker Engine with Compose v2, OpenSSL, and `tar`. Extract
the server archive into a permanent directory:

```bash
sudo install -d -m 0750 /opt/saccadia-remote
sudo tar -xzf SaccadiaRemote-Server-0.4.172-linux-x64.tar.gz \
  -C /opt/saccadia-remote
```

Install Coordinator and the first local Edge. Replace `server.example.com` with the name clients
will actually reach:

```bash
sudo env \
  SACCADIA_COORDINATOR_HOST=server.example.com \
  bash /opt/saccadia-remote/deploy/linux/install.sh
```

Open `https://server.example.com:5443/status` after the containers become healthy and the management
password is configured. A standard installation creates unique persistent certificates locally. A
browser will warn about the management certificate
until the operator configures a public-CA PFX; client and cluster trust do not depend on browser trust
and use authenticated SPKI pins.

Persistent state is under `/var/lib/saccadia-remote`. Re-running the same installer preserves the
database, allowlist, identities, and pins.

## Standalone Linux ServerRelay

ServerRelay is not bundled into the full Server installation. Install it on the same machine or on
another machine that has a stable UDP endpoint reachable by clients. Extract the ServerRelay
archive into its own permanent directory:

```bash
sudo install -d -m 0750 /opt/saccadia-remote-server-relay
sudo tar -xzf SaccadiaRemote-ServerRelay-0.4.172-linux-x64.tar.gz \
  -C /opt/saccadia-remote-server-relay
```

Configure the Coordinator address and public UDP endpoint, then run the relay installer:

```bash
sudo env \
  SACCADIA_COORDINATOR_HOST=server.example.com \
  SACCADIA_RELAY_PUBLIC_HOST=relay.example.com \
  SACCADIA_RELAY_PUBLIC_PORT=5001 \
  SACCADIA_RELAY_LIMIT_MBPS=50 \
  bash /opt/saccadia-remote-server-relay/deploy/linux/install-server-relay.sh
```

`SACCADIA_RELAY_LIMIT_MBPS` is an aggregate cap for this relay. Use `0` for unlimited traffic or a
positive value such as `50` for a fixed Mbit/s ceiling. The relay prints its fingerprint after
installation. Register that fingerprint on the Coordinator together with the relay source IP
address or CIDR before the relay can be offered to clients:

```bash
sudo docker compose -f /opt/saccadia-remote/deploy/linux/compose.yaml exec server \
  dotnet /app/SaccadiaRemote.ServerService.dll server-relay add \
  <relay-fingerprint> <relay-source-ip-or-cidr> \
  --note "Primary server relay"
```

Use the protected status page or the local `server-relay` console commands to inspect, edit,
enable, disable, or delete registered server relays. Server relays are offered only after ordinary
client relay paths are unavailable.

## Additional Linux Edge

Extract the Linux Edge archive into `/opt/saccadia-remote` on the Edge machine. Obtain the
Coordinator cluster pin from its management status page, then run:

```bash
sudo env \
  SACCADIA_EDGE_NODE_ID=edge-2 \
  SACCADIA_EDGE_SIGNALLING_HOST=edge2.example.com \
  SACCADIA_COORDINATOR_HOST=server.example.com \
  SACCADIA_COORDINATOR_CLUSTER_PIN=<coordinator-cluster-pin> \
  bash /opt/saccadia-remote/deploy/linux/install-edge.sh
```

Read `https://127.0.0.1:5100/api/status` on that Edge and copy its `clusterPublicKeyPin`,
`signallingWebSocketUrl`, and `signallingTlsPublicKeyPin`. On the Coordinator machine register the
exact tuple:

```bash
sudo bash /opt/saccadia-remote/deploy/linux/register-second-edge.sh \
  edge-2 \
  <edge-cluster-pin> \
  wss://edge2.example.com:5100/ws \
  <edge-signalling-pin>
```

## Windows full node

Requirements: 64-bit Windows, an elevated PowerShell session, and the .NET 8 ASP.NET Core Runtime
x64. Extract the full Server ZIP into a permanent location such as `C:\SaccadiaRemote\Server`;
Windows services run the bundled Coordinator and Edge binaries from that directory.

Unblock the extracted files and install Coordinator and the first local Edge:

```powershell
Set-Location C:\SaccadiaRemote\Server
Get-ChildItem -Recurse -File | Unblock-File
.\Install.ps1 -CoordinatorHost server.example.com
```

The full-node installer reads the newly generated Coordinator and Edge pins locally, adds the
complete Edge tuple to the allowlist, restarts Coordinator, and waits for Edge and admission
readiness. No manual JSON editing is required.

## Standalone Windows ServerRelay

Install the separate Windows ServerRelay package into a permanent directory such as
`C:\SaccadiaRemote\ServerRelay`. Run the installer from an elevated PowerShell session with the
Coordinator address and relay public endpoint:

```powershell
Set-Location C:\SaccadiaRemote\ServerRelay
Get-ChildItem -Recurse -File | Unblock-File
.\Install.ps1 `
  -CoordinatorUrl https://server.example.com:5000 `
  -PublicHost relay.example.com `
  -RelayPort 5001
```

The installed relay can print its local fingerprint with the `-fingerprint` startup option. Add the
fingerprint and observed relay source IP/CIDR on the Coordinator through the protected status page
or the local server-relay administration command before the Coordinator can use it.

## Additional Windows Edge

Install the separate Windows Edge package into a permanent directory such as
`C:\SaccadiaRemote\Edge` on another machine. First read the Coordinator cluster pin from the
elevated local status command:

```powershell
$status = .\Coordinator\SaccadiaRemote.ServerService.exe status
$coordinatorPin = ($status | Select-String '^clusterPublicKeyPin: ').Line.Split(': ', 2)[1]
```

Then install the additional Edge:

```powershell
Set-Location C:\SaccadiaRemote\Edge
Get-ChildItem -Recurse -File | Unblock-File
.\Install.ps1 `
  -NodeId edge-2 `
  -SignallingWebSocketUrl wss://edge2.example.com:5100/ws `
  -CoordinatorAddress https://server.example.com:7000 `
  -CoordinatorClusterPublicKeyPin $coordinatorPin
```

Read the additional Edge pins and register their exact binding on the Coordinator machine:

```powershell
$edge = curl.exe -ks https://127.0.0.1:5100/api/status | ConvertFrom-Json

Set-Location C:\SaccadiaRemote\Server
.\Register-Edge.ps1 `
  -NodeId $edge.nodeId `
  -ClusterPublicKeyPin $edge.clusterPublicKeyPin `
  -SignallingWebSocketUrl $edge.signallingWebSocketUrl `
  -SignallingTlsPublicKeyPin $edge.signallingTlsPublicKeyPin
```

The registration script updates the complete trusted tuple atomically, retains a previous protected
configuration, and restarts Coordinator. Edge reconnects automatically.

## Client installers

Client installers are not universal downloads in this repository. Coordinator builds them from
bundled templates for its own installation and inserts its bootstrap URL, instance ID, required
version, and signalling TLS pin. Its download page provides Windows x64/x86 MSI, Linux x64
tar.gz, and Android ARM64 APK packages. Linux includes installation and uninstallation scripts and requires .NET 8;
Windows requires the matching .NET 8 Desktop Runtime. Follow the runtime installation link
shown by the installer when a dependency is missing.

The bundled Windows/Linux clients include ten interface languages. There is no language
selection in the installer: the GUI saves the system UI language on first launch, and users
can change it later in Settings. The language-change restart affects only the GUI and opens
its main window; it does not restart host or relay services. GUI-owned sessions may be interrupted.

If Coordinator is deliberately rotated to a different public key, the package fingerprint changes
and the next package request rebuilds the MSI with the new pin. Same-key certificate renewal does
not rebuild it because client trust has not changed.

The client MSI displays the as-is disclaimer and requires acceptance in normal interactive setup.
An unattended installation must explicitly set `SACCADIA_ACCEPT_DISCLAIMER=1`; otherwise Windows
Installer stops before making changes.

The Windows client MSI automatically adds Microsoft Defender process exclusions for the exact
installed Client, HostService, and RelayService executable paths. It preserves existing exclusions
and removes only entries it created during uninstallation. These exclusions reduce Defender
inspection for the three processes; folders and system-wide protection settings are unchanged.
If Defender or a managed policy rejects the change, installation continues and records it in the MSI log.

### Android packaging and updates

Android packaging requires server version **0.4.153** or later. Older server
distributions do not include the Android template or APK builder.

The server distribution includes an `android-arm64` APK template and `package.json`
under `client-templates`, using the same catalog as Windows and Linux. Coordinator
replaces the APK’s public first-run settings with this installation’s bootstrap URL,
instance ID, and signalling public-key PIN, then signs and caches the final package.
The APK requires Android 8 or later. See the [Android user guide](ANDROID-USER-GUIDE.md).

Since 0.4.167, every server, including the official service, uses an Android
package name derived from its persistent ServerInstanceId:
`com.saccadia.remote.s` followed by the first 32 lowercase hexadecimal characters
of SHA-256 over the UTF-8 instance ID. Preserve both ServerInstanceId and the APK
signing identity during migration. Different servers' APKs can coexist. The separate
official Google Play edition uses `com.saccadia.remote` and updates through Google Play.
Starting with 0.4.168, update metadata includes `androidPackageName`; an Android client
hides and refuses updates when that field is absent or differs from its installed name.

Each server keeps a persistent Android signing identity in
`data/android-signing/signing.p12` and its password in `data/android-signing/password`,
relative to the configured server storage root. The identity is generated when the
first Android package is prepared. Both files are private, belong to the server
service account, and must be retained together during backup and migration. Never
include them in public distributions. Replacing this identity prevents existing
Android installations from accepting ordinary updates. It is separate from TLS
certificates so TLS renewal does not change the APK signature.

The Linux Coordinator image includes the required JRE, `keytool`, `apksigner`, and
`zipalign`. On a Windows Coordinator, install a JDK and Android SDK Build Tools,
and configure these service environment variables if the tools are not on PATH:
`SACCADIA_JAVA_PATH` (java.exe), `SACCADIA_KEYTOOL_PATH` (keytool.exe),
`SACCADIA_APKSIGNER_PATH` (Build Tools’ lib/apksigner.jar), and
`SACCADIA_ZIPALIGN_PATH` (zipalign.exe). Restart Coordinator after configuring them.
The server does not need the .NET Android workload to personalize an existing template.

Release publishing builds the template with .NET SDK 10 and its Android workload,
Android SDK 36, and JDK 21. Select that SDK with `-AndroidDotNet` in the release/deploy
scripts or the `SACCADIA_ANDROID_DOTNET` environment variable. The portable server
and desktop projects keep their existing .NET 8 targets. No APK signing keys are
needed on the build machine: signing happens on the deployed server.

Android update checks use `/api/client-packages/android/arm64`, the existing controlled
download queue, and the package SHA-256. The app verifies the file and its signing
identity before asking Android to install it. The user confirms installation through
Android’s system UI. Updating server templates uses the normal deployment process;
restart Coordinator so its catalog loads the new manifest.

From 0.4.172, Android clients with Google Play services retain a minimal ID recovery
credential through Block Store after successful authentication. The signing key
is recovered before registration and is never uploaded to Coordinator. The numeric
ID is scoped to its server; other servers can recognize the same recovered public
key through ordinary authentication. The bundled privacy policy describes this
processing; operators using a custom policy should update their disclosure.
Recovery after full uninstall was tested with Android backup disabled on one phone;
device and Play services compatibility still applies. This does not back up recent
connections, passwords or other settings.

## Certificates, renewal, and migration

Installation-generated Coordinator and Edge certificates are renewed automatically on the same key,
so their pins remain stable. Public-key rotation is a separate coordinated operation. Operator PFX
files remain the operator's responsibility.

Before moving a Coordinator, export its protected migration bundle and preserve its domain, identity,
configuration, allowlist, and database together. Linux provides `export-server.sh` and
`import-server.sh`; the Windows Server package includes `Export-Server.ps1` and `Import-Server.ps1`.

## Operational checks

After installation verify:

- management status reports Coordinator healthy;
- every configured Edge is connected and ready;
- client admission is `ready`;
- every required standalone server relay is registered, enabled, and online;
- session endpoint logs show the Coordinator-selected `loopback`, `lan`, or `punching` mode and
  exact endpoint; a failed LAN/loopback leg is replaced rather than silently changing route class;
- relay-pool `failedCooldown`, session-pool `underfilled`, and pending relay deliveries return to
  zero after startup or a relay replacement;
- the client download page can return Windows x64/x86, Linux x64, and Android ARM64 packages;
- backups and migration restoration have been tested before relying on remote-only access.

Keep a separate administration path for any machine where loss of remote access could cause harm or
expense.

## Public privacy policy

Each Coordinator serves `/privacy` without signing in, on its management HTTPS
listener and its bootstrap HTTPS listener. Client settings open the connected
server's authenticated management origin; before connection they use the selected
bootstrap HTTPS origin. The browser validates the certificate normally. A public
store listing should use the management URL with a publicly trusted certificate.

The standard `wwwroot/privacy-template.html` is anonymous and contains no operator
personal data. To publish your own contact, create UTF-8 `data/privacy-contact.txt`
under the persistent Coordinator storage root. On the standard Linux deployment
the host path is `/var/lib/saccadia-remote/server/data/privacy-contact.txt`; make it
readable by UID 1654. Windows uses the configured persistent server storage root.
The contact is HTML-encoded and read on every request, so no restart is necessary.

If your processing or retention practices differ, publish your full standalone
UTF-8 HTML policy as `data/privacy.html`. The optional `{{OPERATOR_CONTACT}}` token
is replaced with the contact paragraph. This page disallows scripts, external
resources and forms. These operator files survive updates; keep them out of generic
distribution archives. The operator is responsible for an accurate policy,
contact channel and actual data-retention and deletion procedures.
