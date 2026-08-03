# Why Saccadia Remote Can Be Free

Saccadia Remote is intended to be usable without a subscription, paid account, advertising
profile, or sale of session data.

The project is private-source. This public repository explains the product, its architecture,
privacy boundary, and self-hosting model; it does not publish the application source code or grant
open-source rights. Released packages are governed by the terms supplied with them.

## A different server cost model

Remote desktop video and audio can use a great deal of bandwidth. A conventional service that
carries every session through its own central relay fleet must pay for all of that traffic.

Saccadia Remote separates connection setup from session delivery:

- the central service handles small connection messages;
- online devices use compact connection state;
- encrypted media prefers direct or participating relay paths;
- the central fallback relay is used when other paths are unavailable;
- the server does not decode, record, or analyze session contents;
- additional Edge servers can spread connection handling across machines.

An online but idle device therefore costs much less to support than a continuously relayed remote
desktop session. Keeping expensive media processing away from the central service reduces both
operating cost and the amount of private data exposed to infrastructure.

## What free means

The standard project is intended to provide:

- client use without a subscription;
- self-hosting without a per-device server fee;
- no paid account requirement for basic operation;
- no advertising or session-content monetization requirement.

## What free does not mean

Someone still pays for computers, electricity, Internet access, domains, optional hosting,
backups, and administration. Heavy use of a fallback server relay can also create bandwidth cost.

Free use does not promise permanent availability of a public service, guaranteed support, or
compatibility with every network and device. The software is provided as-is.

## Why the design is publicly explained

Users should not have to rely on a vague statement that "the server is trusted." The public
documentation explains what servers and relays do, what metadata they may see, why they cannot
normally read session contents, and which risks remain.

Publishing these boundaries helps users make an informed choice without publishing private source
code or operational details of a live service.
