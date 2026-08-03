# How Saccadia Remote Works

Saccadia Remote separates connection setup from the remote desktop data itself. This separation is
the main reason the service can be both private and inexpensive to operate.

## The connection service

When the application starts, it contacts a Coordinator. The Coordinator recognizes the device and
directs it to an available Edge server. The Edge keeps the small signalling connection used to
start sessions, check availability, and arrange relay paths.

The Coordinator remains responsible for deciding which registered device is online and whether a
connection can be prepared. Edge servers spread long-lived connections across machines. Adding an
Edge therefore adds connection capacity without placing all client connections on one central
process.

Neither Coordinator nor Edge is a remote desktop viewer. They do not decode the screen, play the
sound, inspect the clipboard, or open transferred files.

```mermaid
flowchart LR
    A["Your device"] --> B["Connection service"]
    B --> C["Available Edge"]
    A <-->|"Encrypted session data"| D["Remote device"]
```

## Relay paths

Devices on the Internet cannot always contact each other directly. Firewalls, routers, mobile
networks, or provider restrictions may require another device or server to forward packets.

A relay receives an already encrypted packet and forwards it. It does not receive the key needed
to decrypt the session. The packet contents remain protected whether the path uses a participating
client relay or the fallback server relay.

Saccadia Remote can prepare more than one path for a session. If one route becomes slow or
unavailable, another route can be used without treating the relay as a trusted reader of the data.
Relay use is limited by configuration; an online computer is not intended to become an unlimited
public traffic server.

## What is stored centrally

The server keeps the minimum registration and connection information needed to operate the
service, such as a public device identity and recent availability. Current connections and relay
assignments are short-lived operational state.

The server does not store remote desktop recordings, screenshots, audio, keystrokes, clipboard
contents, chats, or transferred files as part of normal signalling.

## Why the central infrastructure stays small

Traditional relay services may carry every screen and audio byte through their own servers.
Saccadia Remote normally keeps that traffic away from the central connection service. An idle
online device mainly needs a small signalling connection and a compact online record.

This means that connection capacity can be expanded by adding Edge servers, while expensive media
bandwidth remains distributed. The fallback server relay is available when other paths cannot be
used, but it is not the preferred route for every session.

For the security boundary behind this design, read [Security](SECURITY.md). For information about
data visible to operators, read [Privacy](PRIVACY.md).
