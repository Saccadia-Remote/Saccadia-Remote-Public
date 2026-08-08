# Server and Edge installation guide

This guide is for administrators who want to run a private Saccadia Remote installation. The
application source code is not included. The packages contain published service binaries,
installation scripts, configuration templates, and third-party notices.

## Downloads

Current package version: **0.2.95**.

| Role | Operating system | Package |
|---|---|---|
| Server full node | Linux x64 | [Download](packages/SaccadiaRemote-Server-0.2.95-linux-x64.tar.gz) |
| Edge only | Linux x64 | [Download](packages/SaccadiaRemote-Edge-0.2.95-linux-x64.tar.gz) |
| Server full node | Windows x64 | [Download](packages/SaccadiaRemote-Server-0.2.95-windows-x64.zip) |
| Edge only | Windows x64 | [Download](packages/SaccadiaRemote-Edge-0.2.95-windows-x64.zip) |

Verify a downloaded archive against [SHA256SUMS](packages/SHA256SUMS) before extracting it.

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
| 5001 | UDP | Low-use fallback relay |

Only management 5443 is intended for a browser. Port 5000 is client bootstrap, 7000 is cluster mTLS,
and long-lived client WebSockets terminate directly on Edge.

## Protect the detailed status page

The overview page and its aggregate `/api/overview` data remain public. The detailed `/status` page
and `/api/status` require one administrator password; detailed topology, resources, certificates,
and protection state are not part of the public response. Downloads remain public, and `/health`
exposes only minimal liveness for service orchestration. Configure the password before opening the
status page. Linux full-node commands run inside the Coordinator container:

```bash
cd /opt/saccadia-remote
sudo docker compose -f deploy/linux/compose.yaml exec server \
  dotnet /app/SaccadiaRemote.ServerService.dll status-password set
```

Windows full-node commands require an elevated terminal:

```powershell
.\Coordinator\SaccadiaRemote.ServerService.exe status-password set
```

Input is hidden and confirmed. Automation may pipe a secret only with explicit `--password-stdin`;
`--generate` prints one strong password once. Use `status-password status` to inspect whether a
password is configured and `status-password reset` to remove it. Reset immediately invalidates all
browser sessions and blocks detailed browser status until a new password is set. The stored file
contains a salted password verifier, never the plaintext password.

Login is limited globally to five attempts in a rolling 60-second interval, regardless of source IP.
If an attacker occupies that window, inspect the same complete status snapshot without HTTP:

```bash
sudo docker compose -f deploy/linux/compose.yaml exec server \
  dotnet /app/SaccadiaRemote.ServerService.dll status --watch
```

```powershell
.\Coordinator\SaccadiaRemote.ServerService.exe status --watch
```

The local status endpoint is restricted to the service identity/administrators and does not accept a
password. Press Ctrl+C to stop watch mode.

## Linux full node

Requirements: x86-64 Linux, root access, Docker Engine with Compose v2, OpenSSL, and `tar`. Extract
the server archive into a permanent directory:

```bash
sudo install -d -m 0750 /opt/saccadia-remote
sudo tar -xzf SaccadiaRemote-Server-0.2.95-linux-x64.tar.gz \
  -C /opt/saccadia-remote
```

Install Coordinator, the first Edge, and the fallback relay. Replace `server.example.com` with the
name clients will actually reach:

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
Windows services run the bundled Coordinator, Edge, and fallback relay binaries from that
directory.

Unblock the extracted files and install Coordinator, the first local Edge, and fallback relay:

```powershell
Set-Location C:\SaccadiaRemote\Server
Get-ChildItem -Recurse -File | Unblock-File
.\Install.ps1 -CoordinatorHost server.example.com
```

The full-node installer reads the newly generated Coordinator and Edge pins locally, adds the
complete Edge tuple to the allowlist, restarts Coordinator, and waits for Edge, admission, and relay
readiness. No manual JSON editing is required.

## Additional Windows Edge

Install the separate Windows Edge package into a permanent directory such as
`C:\SaccadiaRemote\Edge` on another machine. First read the Coordinator cluster pin from the
elevated local status command:

```powershell
$status = .\Coordinator\SaccadiaRemote.ServerService.exe status
($status | Select-String '^clusterPublicKeyPin: ').Line.Split(': ', 2)[1]
```

Then install the additional Edge:

```powershell
Set-Location C:\SaccadiaRemote\Edge
Get-ChildItem -Recurse -File | Unblock-File
.\Install.ps1 `
  -NodeId edge-2 `
  -SignallingWebSocketUrl wss://edge2.example.com:5100/ws `
  -CoordinatorAddress https://server.example.com:7000 `
  -CoordinatorClusterPublicKeyPin $coordinator.clusterPublicKeyPin
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

Client MSI packages are not universal downloads in this repository. Coordinator builds them from
bundled templates for its own installation and inserts its bootstrap URL, instance ID, required
version, and signalling TLS pin. Download the resulting Windows x64 or x86 client from the installed
server's management page.

If Coordinator is deliberately rotated to a different public key, the package fingerprint changes
and the next package request rebuilds the MSI with the new pin. Same-key certificate renewal does
not rebuild it because client trust has not changed.

The client MSI displays the as-is disclaimer and requires acceptance in normal interactive setup.
An unattended installation must explicitly set `SACCADIA_ACCEPT_DISCLAIMER=1`; otherwise Windows
Installer stops before making changes.

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
- fallback relay is online;
- the client download page can build and return both required Windows packages;
- backups and migration restoration have been tested before relying on remote-only access.

Keep a separate administration path for any machine where loss of remote access could cause harm or
expense.
