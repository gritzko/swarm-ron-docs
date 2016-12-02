# Swarm replicas

The Swarm core itself assumes the role of a *peer* as a node that keeps the full operation log of the database.
Peers can serve *clients* that have either a full log or (most often) an arbitrary subset of objects.
For the op forwarding logic, the nature of peers is irrelevant.
It may be a centrally administered set of geo-distributed servers or a network with [dynamic membership](peerage.md).
[Base64x64 replica ids](stamp.md) assume a hierarchic *naming scheme* consisting of up to four components (chunks), denoted by chunk lengths, like:
* `0280` (2 chars for a peer id chunk, 8 for client id chunk)
* `0262` (2 for peers, 6 for clients, 2 for a client's sessions)
* `1261` (1 char for [primus id](peerage.md), 2 for peers, etc)

Tailing chunks can be unfilled (zero) and thus omitted.
A filled id chunk is never zero.
For example, in the `0262` scheme, a client is identified with a 0+2+6=8 char long Base64x64 number, but that id does not correspond to any actual replica, as actual replicas have non-zero session ids.
In the same scheme, a peer has an identifier of 2 chars, and that identifier corresponds to an actual replica with a certain [op](op.md) arrival order.

## Primuses

Primuses is an optional layer of "true" peers, mostly meaningful in dynamic-membership networks.
The term is borrowed from the Latin formula `primus inter pares`.
Primuses must all recognize each other, e.g. cryptographically sign each other's signatures.
Any primus MUST be able to connect to any other primus any time, should the need arise.
Primuses may introduce new peers that can connect to any other peers.
The all-know-all pattern has its scalability limits, hence primus id chunk is 0-2 chars.

## Peers

Peers possess a full replica of a database and a full log.
A peer MUST have at least one live connection to some other peer.
Peers are generally not supposed to be offline, unless in the case of catastrophic failure of the underlying infrastructure.
Peers MAY connect to more peers and MAY accept connections from other peers.
Peers can serve clients.
Peers are the most flexible joint in this tree; a peer id chunk may be 0 to 10 chars.

## Clients

A client has a partial replica of a database and a filtered log, provided by some peer.
Clients can only synchronize with their upstream peers.
Although there is a special technique to [move a client replica](handover.md) to a different peer, that is an exceptional situation.
Note that clients can have the same id, when connected to a different peer, e.g. Xclient and Yclient, depending on [database options](options.md).
A client chunk length is 0 to 8 chars (more than enough to number every person on the planet and every their hair).

## Sessions

The same client (e.g. a person) can have multiple devices or multiple browser sessions.
A client can open multiple sessions, all connected to the same peer.
Although all those replicas have the same access rights, they are still different replicas with their own local op logs.
Hence, they have different replica ids.
As all the client's sessions are connected to the same peer, it is possible to recycle session ids by invalidating long-unseen sessions.
1-2 chars is enough for a session id chunk.

## Compound ids

Replica ids are normally used as the second part of a [Lamport timestamp](stamp.md). Hence, they are part of stamps and object ids. Also, replica ids are a part of stamp-like identifiers which are not stamps per se:

* db replica id has a form of `dbname+replicaid` (this one is used when either a client or a server has multiple databases open)
* connection id has a form of `timestamp+replicaid` (the timestamp comes from the upstream, while the replica id refers to the downstream)
