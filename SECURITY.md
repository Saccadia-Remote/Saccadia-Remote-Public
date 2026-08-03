# Security and Encryption

Saccadia Remote is designed around end-to-end encryption. Remote session data is protected before
it leaves one participating device and is decrypted only after it reaches the other.

This is a technical boundary, not only a promise by a server operator. The standard server and
relay components do not receive the session keys and do not include a normal way to decrypt a
session.

## What is protected

The encrypted session includes:

- screen video and sound;
- keyboard, mouse, and cursor information;
- clipboard data;
- chat messages;
- transferred files;
- permissions and session-control messages.

Changing an encrypted packet causes authentication to fail, so damaged or forged session data is
discarded instead of being accepted as genuine.

## Password-protected connections

The host password is not sent to Coordinator, Edge, or relay servers. The two devices use a
password-authenticated exchange to prove that they know the same password and to establish fresh
session keys. A captured network exchange does not provide the server with a reusable password.

One-time passwords follow the same principle and are replaced after successful use. When a user
chooses saved authorization, the participating devices keep a separate protected secret and prove
it again during a future connection. Knowing only the public device ID is not enough.

## Passwordless manual connections

A passwordless connection still encrypts its traffic, so ordinary relays and passive network
observers cannot read the session. However, the signalling service helps the two devices exchange
the information used to create their keys. An actively compromised signalling service is therefore
inside the trust boundary for this mode.

Use a host password when protection from that situation is important. Manual approval remains
useful for convenience, but it should not be described as having exactly the same trust boundary
as a password-authenticated connection.

## Device and server identity

Each client creates its own device identity locally. The server stores the public part and requires
the device to prove that it still owns the private part.

Clients also verify the identity of the server they were configured to use. Server installations
have their own persistent identities; the product does not distribute one shared server private
key. A server can be moved with its identity through the supported protected backup process.

## What relays and servers can still observe

Encryption hides content, but it cannot hide everything needed to deliver network traffic. An
operator may see public device identifiers, IP addresses, connection times, online state, traffic
size, and routing information. These details are metadata and can still be sensitive.

Saccadia Remote is a remote access product, not an anonymity network.

## Protection against abandoned network work

Network-facing components limit pending connections, unfinished authentication, relay state, and
incomplete packets. Old or invalid work expires instead of being allowed to consume memory
forever. Operational limits are configurable by the server administrator and are intentionally not
part of this public user overview.

These protections reduce accidental and malicious resource exhaustion. They cannot guarantee
availability during a large network attack or an infrastructure outage.

## What encryption cannot protect against

End-to-end encryption does not protect against:

- malware or an attacker already controlling either participating device;
- a modified client application that records data before encryption or after decryption;
- installing a package from an untrusted publisher or server operator;
- authorizing the wrong person or granting unnecessary permissions;
- traffic analysis based on addresses, size, and timing;
- denial of service, packet dropping, or loss of Internet access;
- vulnerabilities in the operating system, drivers, codecs, or other dependencies.

Use trusted installation packages, keep devices updated, and grant only the permissions required
for a session.

## Diagnostics and vulnerability reports

Diagnostic logging is disabled by default. When enabled, logs may contain device identifiers, IP
addresses, timings, route information, and errors. They must not contain passwords, private keys,
or decrypted session contents. Review logs before sharing them.

Do not publish an exploitable vulnerability or private user data in a public issue. Use the
repository's private security reporting channel when available and include the affected component,
observed behavior, and reproduction steps without including live credentials or private keys.
