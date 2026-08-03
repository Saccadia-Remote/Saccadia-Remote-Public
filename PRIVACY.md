# Privacy

Saccadia Remote does not need to read the contents of a remote session in order to connect two
devices. In the standard application, session contents are encrypted end to end and are not
available in plaintext to Coordinator, Edge, or relay servers.

The project does not use screen contents, sound, clipboard data, chats, input, or transferred files
for advertising, behavioral profiling, sale of data, or AI training.

## What the service can see

Every Internet service needs some information to deliver traffic. Saccadia Remote infrastructure
may process:

- a public device identifier and public identity key;
- whether a device is online or was seen recently;
- IP addresses and connection times;
- the Edge and relay paths used for a connection;
- traffic size, timing, and operational error information.

This metadata can reveal that devices communicated and when. It should not be confused with the
encrypted contents of the session.

## What the service does not store as session history

Normal signalling does not create server-side copies of:

- screen recordings or screenshots;
- audio recordings;
- keystrokes or mouse actions;
- clipboard contents;
- chat history;
- transferred files.

The server keeps compact registration information needed to recognize a device and operate the
service. Temporary online, session, and relay information is kept only for operational purposes.

## Optional Wake-on-LAN information

Wake-on-LAN is an explicit exception to the minimal registration record. If a user enables it, the
server stores the network information needed to request that a device be woken. Disabling the
feature removes that stored profile.

Leave Wake-on-LAN disabled if this convenience is not required.

## Diagnostics

Diagnostic logging is local and disabled by default. If a user or administrator enables it, logs
may contain public device IDs, IP addresses, connection timings, route identifiers, sizes, and
errors. Logs are not automatically sent to an advertising or analytics service by the standard
project.

Operational metadata can still be sensitive. Review logs before sharing them and remove old logs
when they are no longer needed.

## Self-hosting

A self-hosting operator controls the registration database, server identity, configuration,
backups, and optional logs. Running the standard server does not by itself provide session keys,
but the operator can see the metadata described above.

An operator may also distribute modified client software. End-to-end encryption cannot protect a
user from a malicious client application, so obtain installation packages from a publisher or
self-hosting administrator you trust.

## Practical privacy choices

- Use a trusted installation package.
- Configure a host password when you need the stronger authenticated connection mode.
- Enable only the remote permissions required for the task.
- Leave optional data features disabled when they are not needed.
- Keep diagnostic logging off during normal use.
- Self-host when you want direct control over registration metadata and server logs.
