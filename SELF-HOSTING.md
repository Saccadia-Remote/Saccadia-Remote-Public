# Self-hosting

Saccadia Remote can use the public service or infrastructure controlled by a user or organization.
Self-hosting is supported for people who want direct control over availability, registration
metadata, server identity, logs, and update timing.

## What a server installation contains

A complete installation has three roles:

- **Coordinator** recognizes devices and arranges connections;
- **Edge** holds client signalling connections and can be added to spread connection load;
- **fallback relay** forwards encrypted session packets when other paths are unavailable.

These roles can run together for a small installation. Additional Edge machines can be added when
more connection capacity or separation between locations is needed. Server roles support Linux and
Windows.

## Server identity

Each installation uses its own persistent cryptographic identity. It can use a certificate
provided by the operator or create a unique identity during installation. The product does not
ship one shared server private key.

Clients remember the identity of the server they were configured to trust. Server identity files,
configuration, and the registration database must therefore be backed up together.

The supported protected export/import process can move a server while preserving its identity.
When the same public name and identity are retained, installed clients do not need to accept an
unexpected server key. A planned change to a different key must be coordinated explicitly.

Detailed downloads, installation commands, service locations, Edge registration, certificate
behavior, and network settings are available in the
[Administrator installation guide](ADMINISTRATOR-GUIDE.md).

## What self-hosting changes

Self-hosting gives the operator control over:

- where registration metadata is stored;
- where Edge servers and the fallback relay run;
- certificate and backup handling;
- log retention and monitoring;
- software update timing;
- network access policy.

It does not make session contents visible to the standard server components. It also does not
remove the need to trust endpoint applications and operating systems.

## Operator responsibilities

A self-hosting operator is responsible for protecting server keys, configuration, backups,
databases, and logs; restricting administrative access; applying security updates; monitoring the
service; and testing recovery.

Keep a separate access method for machines where loss of remote access would be costly. The project
does not provide a guaranteed service level or administration service for self-hosted deployments.
