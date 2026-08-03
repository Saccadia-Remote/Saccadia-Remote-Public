# Saccadia Remote

Saccadia Remote is a free remote desktop application for connecting to a computer from another
location. It is designed so that the servers which help devices find each other do not need access
to the contents of a remote session.

The screen, sound, keyboard and mouse input, clipboard, chat, and transferred files are encrypted
on one participating device and decrypted only on the other. Coordinator, Edge, and relay servers
handle connection setup or forward encrypted packets. They do not receive the session keys needed
to read those packets.

## Why this is safer than trusting a relay

Think of a relay as a courier carrying a locked parcel. It needs enough information to deliver the
parcel, but it does not have the key to open it. Using several relay paths can make a connection
more reliable without giving any relay access to the contents.

The infrastructure may still see ordinary network metadata, such as IP addresses, connection
times, and traffic size. Saccadia Remote protects session contents; it is not an anonymity service.

## Passwords stay on the participating devices

A remote-access password is not sent to the signalling server. The two devices use a
password-authenticated exchange to prove that they know the same password and to create session
keys. Saved authorization also relies on a secret held by the two devices, not only on a client ID
stored by the server.

For the strongest protection, configure a host password. A passwordless connection that relies
only on manual approval has to trust the signalling service to introduce the correct devices. This
limitation is explained in [Security](SECURITY.md).

## Why the service can remain free

Normal remote desktop video and audio are not processed or stored by a large central server. The
central infrastructure mainly handles small connection messages, while encrypted session traffic
uses direct or distributed relay paths whenever possible. This greatly reduces the server capacity
and bandwidth needed for idle users and ordinary sessions.

Free use does not mean that hosting, Internet access, electricity, or administration have no cost.
It means that the standard project does not require a subscription, advertising profile, or sale
of session data.

## Run your own server

Organizations and individual users can run their own Coordinator, Edge, and fallback relay. Server
roles support Linux and Windows. A small installation can place the roles together, while a larger
installation can add Edge servers to spread connection handling across machines.

Each installation uses its own server identity. The supported backup and migration process can
preserve that identity when a server is moved. See [Self-hosting](SELF-HOSTING.md).

## Important limits

End-to-end encryption does not protect a device that is already infected, an untrusted or modified
client application, or a person who was deliberately granted access. Install the application only
from a source you trust, protect the host with appropriate authorization, and keep another access
method for systems where remote availability is critical.

Saccadia Remote is under active development and is provided as-is, without a warranty or guaranteed
service level. See [Disclaimer](DISCLAIMER.md).

## Documentation

- [How the system works](ARCHITECTURE.md)
- [Security and encryption](SECURITY.md)
- [Privacy](PRIVACY.md)
- [Self-hosting](SELF-HOSTING.md)
- [Why the project can be free](FREE-AND-PUBLIC.md)
- [Frequently asked questions](FAQ.md)
- [Disclaimer and user responsibilities](DISCLAIMER.md)
