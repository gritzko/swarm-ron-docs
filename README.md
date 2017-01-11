# The Swarm Protocol 1.4.0 #
[*see on GitBooks: PDF, ebook, etc*](https://gritzko.gitbooks.io/swarm-the-protocol)

Swarm is a Web-scale protocol for distributed data synchronization.
Swarm is envisioned to synchronize structured data, on an unlimited number of devices, in real-time, in presence of failures.

Swarm is end-to-end: it spans to the client side and its complexity is at the *edge*, while the *core* mostly relays messages.
Swarm supports partial datasets at the edge, as the full dataset may turn too big for client devices.
Swarm is eventually consistent (causal consistency, partial order), as no linearization is possible with client devices.
Swarm network topology is super-peer federated.
A cloud of *peers* maintains the full event log and the full state; *clients* subscribe with their peer to receive state snapshots and event feeds of their choosing.
Swarm is reactive and asynchronous; if TCP is a "data pipe" then Swarm is an "event pipe".
Swarm's mission is to seamlessly synchronize data in the background, in real-time, dealing with failures and concurrency, to support its two API abstractions:

1. an asynchronous event pipe (the lower API layer) and
2. automagically synchronized objects (the upper API layer).

Technically, Swarm is based on a partially-ordered log of immutable operations.
Every Swarm event/operation is stamped with a globally unique identifier.
A [Swarm id](id.md) is a 128-bit hybrid timestamp citing the event's time and origin, very much like [UUID v1][uuid].
Those Swarm ids are used to reference any event or entity in the system.
Swarm explicitly versions all the data, so every version is referenced by its Swarm id too (in the local context).
Essentially, Swarm ids solve two problems: naming things and cache invalidation.

In academic terms, Swarm is a [reliable causal broadcast][opbased] with [replicated data types][rdt] on the top.

The Swarm's projected use cases can be broadly divided into three classes:

1. asynchronous incremental change propagation allows for collaborative and real-time apps,
2. data caching and offline writability allows for autonomous client-side replicas (mobile, IoT),
3. change attribution naturally allows for federated multi-tenant databases ran by multiple parties at once.

[2sided]: http://lexicon.ft.com/Term?term=two_sided-markets
[super]: http://ilpubs.stanford.edu:8090/594/1/2003-33.pdf
[opbased]: http://haslab.uminho.pt/sites/default/files/ashoker/files/opbaseddais14.pdf
[cap]: https://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed
[rdt]: TODO
[uuid]: http://tools.ietf.org/html/rfc4122

## Inner parts

Following the TCP/IP analogy, Swarm offers its own stack of progressively higher-level abstractions. That stack can be explained in nine letters:

    API
    RDT
    LOG

At the bottom, there is a log of immutable [operations](op.md).
At the next layer, RDT reducers turn streams of operations into object states.
At the top layer, objects' internal states are turned into idiomatic APIs for a particular language.
We an compare these three layers to IP packets, TCP data streams and POSIX sockets respectively.

### Op log primitives -- ids, ops

Any change is introduced into the system as an [operation](op.md) with a [globally unique id](id.md).
Hence, any entity can be referenced by the id of its genesis op.
An op consists of five parts: four ids and a value.
Four ids are:

1. object id,
2. replicated data type (reducer) id,
3. event/op own id and
4. location id.

Those four ids form an operation's generic "header".
Like in TCP/IP, op routing only needs the header.
The value is either an arbitrary piece of JSON or a *reference* (an object id).
Thus, the value is an op's free-form "body" carrying all the datatype-specific information.

### Data types

On top of the log, replicas can run versioned data structures formally described as [Replicated Data Types](rdt.md).
The main focus is on op-based [CmRDTs](crdt.md#CmRDT).
Swarm can support other constructs as well, such as state-based [CvRDT](crdt.md), logged asynchronous RPC and others, even [crypto coins](coin.md).
An RDT object can be described in the terms of the [state machine replication][smr] model: an *object* has a *state* mutated by inputs (*ops*).
Those ops get delivered to every object's replica, so eventually their states converge.
A data type can also be seen as a *reducer* function that turns a stream of ops into the resulting state.
Swarm can only employ data types that converge despite some possible reordering of concurrent operations (i.e. concurrent ops must commute).

### API

On the top of the RDT layer, there is a language-specific idiomatic API that hides all the op/state/metadata internals from the developer.
The general concept of those APIs is that replicated data types must pretend to be plain simple local objects as much as possible.
All the synchronization must happen automagically.

[smr]: https://www.cs.cornell.edu/fbs/publications/SMSurvey.pdf

## Network topology

Swarm picks the middle way of a super-peer network topology.
Networks that are symmetric by design tend to display unavoidable stratification in practice (mostly, because of economies of scale).
At the same time, explicit stratification greatly contributes to the protocol's practicality.

Swarm *peers* connect to each other in an arbitrary fashion, so the graph is connected most of the time.
Each peer must keep a full database replica.
A *clients* only connects to its *home* peer.
Clients can pick their dataset on object-by-object basis.
Client replicas are still fully autonomous, can cache all the data locally and make writes while offline.

Every Swarm [op](op.md) is [timestamped](stamp.md) and attributed to its origin replica.
Ops are immutable from the moment of creation (e.g. compare that to repeatedly-mutable [OT][ot] ops).
Ops propagate without any [causality](order.md) violations.
Practically, that means a replica can only relay ops in the order they were received (the [*delivery* order](order.md)).

Ideally, a fresh op propagates with the network speed: client -> home peer -> ... -> peer -> client.

[ot]: https://en.wikipedia.org/wiki/Operational_transformation

## Comparisons

Swarm bears strong resemblance to message bus / distributed log systems (Apache Kafka, Facebook Scribe), except it is partially ordered, so it can span to the client side and survive partitions (AP by [CAP][cap]).
The core of Swarm is a replicated log service; it also builds a key-value database on top of that.
Potentially, Swarm network architecture covers the range of use cases from a geo-distributed eventually consistent data store all the way to a [super-peer network][super].
Swarm is neither a linear-log ACID database nor a symmetric peer-to-peer network.


## The math

Swarm  relies on a variety of formal computer science models:

* state machine replication,
* Lamport logical time,
* sequential processes exchanging messages,
* Commutative Replicated Data Types.

An implementation is likely to rely on a combination of TCP's sequential delivery and log-structured database guarantees.

The core contribution of the Swarm protocol is its *practicality* in regard to various trade-offs.
For example, it is possible to implement a CRDT-based database literally, along the definitions, but that will hardly be practical.
Swarm arranges primitives in a way to make metadata overhead acceptable, which is a known hurdle with CRDT-based solutions.
In particular, Swarm avoids the *explicit* use of version vectors.
Those turn untenable when every client device runs its own replica.

Swarm [op format](op.md) is made simple enough to fit the limitations of a key-value storage.
It keeps the overhead low for op-chatty applications.
For example, realtime collaborative text editors are likely to create one op for one keystroke.

A good entry point to start studying the Swarm protocol is its [subscription handshakes](handshake.md).

Use Swarm.


### History

* 2012-2013: started as a part of the Yandex Live Letters project
* 2014 October: becomes a separate project, version 0.3 is demoed (per-object logs and version vectors, not really scalable)
* 2015: version 0.4 is scrapped, the math is changed to avoid any version vector use
* 2016 Feb: version 1.0 stabilizes (no v.vectors, new asymmetric client protocol)
* 2016 May: version 1.1 gets peer-to-peer (server-to-server) sync
* 2016 June: version 1.2 gets crypto (Merkle, entanglement)
* 2016 Nov: version 1.4 gets references, structured values and the unified state/patch/log compressed format
