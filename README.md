# The Swarm Protocol 1.2.1 #
[*see on GitBooks: PDF, ebook, etc*](https://gritzko.gitbooks.io/swarm-the-protocol)

Swarm is a protocol for distributed data synchronization.
Swarm is convergent (eventually-consistent) and spans to the client side.
Technically, it does partially-ordered operation log synchronization.
Swarm explicitly versions the data, every change is stamped with time and origin.
Swarm supports:

* end-to-end real-time incremental sync (op-based),
* partial datasets on the client (of arbitrary selection, no "rooms"/"channels"),
* client-side caching (as the data is versioned, a cache can always be incrementally updated),
* offline work (writes are queued, resubmitted on reconnection),
* shared databases (where every change is attributed),
* and lots of other exciting things.

Swarm network architecture covers the range of use cases from a geo-distributed eventually consistent data store all the way to a [super-peer network][super].
One important use case in the middle of the range is a shared database run by multiple parties (e.g. a [two-sided market][2sided] data exchange scenario).
Another good fit is a real-time collaborative app backend.
Swarm is neither a linear-log ACID database nor a symmetric peer-to-peer network.
Swarm bears strong resemblance to message bus / distributed log systems (Apache Kafka, Facebook Scribe), except it is partially ordered, so it can span to the client side and survive partitions (AP by [CAP][cap]).

Swarm focuses on immutable op log synchronization.
In academic terms, Swarm is a [reliable causal broadcast][opbased] and some replicated data types on top of that.

Swarm synchronizes autonomous writable replicas.

[2sided]: http://lexicon.ft.com/Term?term=two_sided-markets
[super]: http://ilpubs.stanford.edu:8090/594/1/2003-33.pdf
[opbased]: http://haslab.uminho.pt/sites/default/files/ashoker/files/opbaseddais14.pdf
[cap]: https://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed

## Inner parts

Swarm replica architecture can be explained in nine letters:

    API
    RDT
    LOG

At its bottom, Swarm is a replication system for a *partially* ordered log of immutable [operations](op.md).
Partial order means operations may be created concurrently at different replicas, then propagated globally.
Concurrent operations may appear in different orders at different replicas.

On top of the log, replicas can run versioned data structures formally described as [Replicated Data Types](rdt.md).
The main focus is on op-based [CmRDTs](crdt.md#CmRDT).
Swarm can support other constructs as well, such as state-based [CvRDT](crdt.md), logged asynchronous RPC and others, even [crypto coins](coin.md).

An RDT replica can be described in the terms of the [state machine replication][smr] model.
Swarm only uses data types that converge despite some possible reordering of concurrent operations (i.e. concurrent ops must commute).

On the top of the RDT layer, there is a language-specific idiomatic API that hides all the op/state/metadata internals from the developer.
The general concept of those APIs is that replicated data types must pretend to be plain simple local objects as much as possible.
All the synchronization must happen automagically.

[smr]: https://www.cs.cornell.edu/fbs/publications/SMSurvey.pdf

## Network topology

Swarm picks the middle way of a super-peer network topology.
Networks that are symmetric by design tend to display unavoidable stratification in practice (e.g. consider BitCoin miners).
Such a dynamics likely owes to Adam Smith like laws: economies of scale and specialization provide too much benefit to be ignored.
Hence, the ideal of a fully symmetric peer-to-peer network appears to be not worth pursuing.
Meanwhile, ACID databases have very clear scalability thresholds.
Also, the classic ACID database design normally assumes a single (institutional) user.
A linear-log system can hardly be ran otherwise.
Consequently, they save on the who-when-why metadata.

Swarm *peers* connect to each other in an arbitrary fashion, the only requirement is that the graph should be connected most of the time.  
Peers must keep a full database replica.
*Clients* only connect to their *home* peers.
Clients can pick their dataset on object-by-object basis.
Client replicas are fully autonomous, can cache all the data locally and make writes while offline.

Every Swarm op is timestamped and attributed to its origin replica.
Ops are immutable from the moment of creation (e.g. compare that to repeatedly-mutable [OT][ot] ops).
Ops propagate without causality violations.
Practically, that means a replica can only relay ops in the order they were received (the *arrival* order).

One may argue that Swarm is neither a true database (no indexes, no query language) nor a true peer-to-peer network ([*peer* admission](peerage.md) is not completely open).
That is certainly a trade-off and there are ways to overcome those issues (e.g. database integrations and open *client* admission).
So, yet another way to describe Swarm is "like Kafka, but partially ordered, so it can work on the client and survive disconnections".
That is the core of it: a replicated log service (and a key-value database on top).

[ot]: https://en.wikipedia.org/wiki/Operational_transformation

## The math

Swarm employs a variety of classic computer science models: state machine replication, Lamport logical time, sequential processes exchanging messages.
It relies on TCP sequential delivery and log-structured database guarantees.

The core contribution of the Swarm protocol is *practicality*.
It is possible to implement a CRDT-based database literally, along the definitions, but that will hardly be practical.
Swarm arranges primitives in a way to make metadata overhead acceptable, a known hurdle in CRDT-based solutions.
In particular, Swarm avoids the *explicit* use of version vectors.
Those turn untenable when every client device runs its own replica.
Swarm [op format](op.md) is made particularly lightweight ([string key](spec.md) - buffer value).
That enables such realtime apps as collaborative text editors where one op is one keystroke.

A good entry point to start studying the Swarm protocol is its [subscription handshakes](handshake.md).

Use Swarm.


### History

* 2012-2013: started as a part of the Yandex Live Letters project
* 2014: becomes a separate project, version 0.3 is demoed in October (per-object logs and version vectors, not really scalable)
* 2015: version 0.4 is scrapped, the math is changed to avoid any version vector use
* 2016: version 1.0 stabilizes by February (no v.vectors, new asymmetric client protocol)
* 2016: version 1.1 gets peer-to-peer (server-to-server) sync in May
