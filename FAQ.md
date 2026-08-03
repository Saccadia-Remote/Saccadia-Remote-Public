# Frequently Asked Questions

## Can the Saccadia Remote server see my screen?

No. The screen is encrypted on the host and decrypted by the viewer. The standard Coordinator,
Edge, and relay components do not receive the session keys needed to read it.

The same applies to sound, keyboard and mouse input, clipboard data, chat, and transferred files.

## Does my remote-access password reach the server?

No. The password remains on the participating devices. They prove that they know the same password
and use that proof to create fresh session keys. The signalling service does not receive a reusable
password or password-equivalent value.

## Is saved authorization based only on my device ID?

No. The two devices keep a separate protected secret and prove it again during a later automatic
connection. A public device ID alone is not enough to reconnect.

## What about a connection without a password?

Its session traffic is still encrypted, so ordinary relays and passive network observers cannot
read it. However, this mode relies more strongly on the signalling service to introduce the correct
devices. Configure a host password when you need the stronger password-authenticated trust model.

## Can a relay read my data?

No. A relay forwards packets that were already encrypted by an endpoint. It can observe network
metadata such as addresses, packet sizes, and timing, but it does not receive the session key.

## Why are several relay paths useful?

Internet routes can fail or slow down independently because of routers, firewalls, wireless loss,
or provider restrictions. More than one available path lets a session recover without trusting a
single relay with plaintext data.

## Is my computer used as an unlimited public relay?

No relay is intended to be unlimited. Relay participation and available bandwidth are controlled
by application and server policy. Disabling participating relay use can reduce connectivity through
difficult networks, where a fallback server relay may then be required.

## What can a server operator see?

The operator may see public device identifiers, IP addresses, connection times, online state,
routing information, and traffic size. Optional diagnostics may contain similar operational
metadata. The operator does not receive normal session contents in plaintext from the standard
client protocol.

## Is diagnostic information uploaded automatically?

Diagnostic logging is local and disabled by default. If it is enabled for troubleshooting, review
the logs before sharing them because connection metadata can still be sensitive.

## Can I run my own server?

Yes. Server roles support Linux and Windows. A small installation can run the required roles
together, and additional Edge servers can be added when more connection capacity is needed.

See [Self-hosting](SELF-HOSTING.md).

## Does every installation share the same server key?

No. A server uses an operator-provided identity or creates a unique persistent identity during
installation. The product does not contain one shared server private key. The supported protected
backup process can preserve an identity when a server is moved.

## Is Saccadia Remote open source?

No. The product source code is private. This repository contains public product and security
documentation so users can understand the design and its limitations. Free use does not grant
source-code modification or redistribution rights.

## Is the software really free?

The standard project is intended to be available without a subscription or paid account. Users and
self-hosting operators still pay their own hardware, Internet, electricity, domain, hosting, and
administration costs.

## Does encryption make remote access completely safe?

No security feature can protect an already compromised device, an untrusted client application, or
a person who was deliberately granted access. Use trusted packages, protect the host, keep systems
updated, and grant only the permissions required.

## Who is responsible if something goes wrong?

Users and self-hosting operators are responsible for lawful use, device security, authorization,
configuration, backups, infrastructure, and the actions they perform or permit. The software is
provided as-is without guaranteed availability or fitness for a particular purpose. See
[Disclaimer](DISCLAIMER.md).
