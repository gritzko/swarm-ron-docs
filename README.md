# The Swarm Protocol 1.1.1-pre #

Swarm is a protocol for partially-ordered-op-log-based synchronization to support a massively replicated eventually consistent database.
Swarm is designed to function both on the server and the client side, hence it supports:

* partial datasets,
* client-side caching,
* offline work and
* real-time incremental sync.

At its core, Swarm is a replication system for a *partially* ordered [operation](op.md) logs.
On top of that, users can run versioned data structures formally described as Replicated Data Types.
The Swarm's main focus is on op-based [CmRDTs](crdt.md#CmRDT).
It can support other constructs as well, such as state-based [CvRDT](crdt.md), logged asynchronous RPC and others.

Swarm puts an accent on practicality.
It arranges primitives in a way to make metadata overhead acceptable, a known hurdle in CRDT-based solutions.
In particular, Swarm avoids the use of version vectors.
Those turn untenable when every client device runs its own replica.
Swarm [op format](op.md) is particularly lightweight to enable such apps as collaborative text editors where one op is one keystroke.

A good entry point to start studying Swarm is its [subscription handshakes](handshake.md).

Use Swarm.


### History

* 2012-2013: started as a part of the Yandex Live Letters project
* 2014: becomes a separate project, version 0.3 is demoed in October (per-object logs and version vectors, not really scalable)
* 2015: version 0.4 is scrapped, the math is changed to avoid any version vector use
* 2016: version 1.0 stabilizes by February (no v.vectors, new asymmetric client protocol)
* 2016: version 1.1 gets peer-to-peer (server-to-server) sync in May
