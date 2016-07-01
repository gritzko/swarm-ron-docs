# Distributed digital currency as an RDT

*Disclaimer.* This text is not put here for the sake of proposing a new crypto currency.
Instead, it pursues four goals:

* give an example of an RDT type, which is not a CRDT,
* give some arguments against distributed linear data structures (ledgers and blockchains),
* give an example of how the entanglement crypto works and
* give a technique for explicit handling of replica inconsistencies.

So, some distributed systems experience scaling difficulties because of their use of a global linearization of all the transactions in the system.
This approach is named ["blockchain"][bitcoin] or ["distributed ledger"][stellar].
The search for a scalable blockchain scheme seems to be the modern day equivalent of both the philosopher's stone and [squaring the circle][circle]: greed-driven pursue of mathematical impossibility.
First of all, the real-world bookkeeping never attempted to linearize all the world's transactions, in any form.
Instead, the classic bookkeeping employs the eventually consistent approach: write it down first, reconcile later.
Real-world ledgers are linear, but only locally and not globally.   
Notably, alternative bookkeeping systems (e.g. [Hawala][hawala]) follow the same pattern: linear local logs, no global linearization.

Those technologies originated well before the Internet.
But, a similar picture is observed in the world of databases.
Highly scalable databases (e.g. Cassandra, Riak) tend to avoid total order.
When a linear-order database (e.g. MySQL) is scaled up, it is typically *sharded*.
In other words, they give up on the global linear order.

Hidden in plain sight is the fact that the total linearization of all transactions is not actually needed for a crypto currency.
The same way, it is not needed by any real-world currency, because unrelated transactions don't need to be ordered.
A linear order is needed for a single *asset* that changes hands.
It is also needed for an *agent* that trades assets.
But not for the system as a whole.

The choice of crypto currencies to implement global linearization can be only explained by their *global consensus* algorithms.
Probably, we should call that global *nonsensus*, because, again, no single actor in the system needs global consensus as such.
Practically, given that we know a coin's owner, the only thing we need to know more is whether that owner gives it to us or to somebody else.
The history and the state of any other asset is completely irrelevant, unless we speak of atomic transactions (exchange of several assets at once), which is a [separate topic][beilis].

Distributed systems need distributed math.

[bitcoin]: https://bitcoin.org
[stellar]: http://stellar.org
[beilis]: http://www.bailis.org/papers/ramp-sigmod2014.pdf
[lamport]: http://research.microsoft.com/users/lamport/pubs/time-clocks.pdf
[circle]: https://en.wikipedia.org/wiki/Squaring_the_circle
[hawala]: https://www.treasury.gov/resource-center/terrorist-illicit-finance/Documents/FinCEN-Hawala-rpt.pdf

## Network topology

> The distributed currency of Ethereum does not depend on any government or bank, but only on Vitalik Buterin.

We generally assume a super-peer network architecture.
A multi-level architecture may not satisfy some, but even de-jure flat networks (e.g. BitCoin) tend to have de-facto stratification (e.g. exchanges and top miners).
Adam Smith-like economies of scale and specialization inevitably lead to the emergence of high-volume high-efficiency nodes.
Complex networks tend to be hub-dominated.
We don't want to argue with the nature.

There are *peer* [Swarm replicas](replica.md) and *client* Swarm replicas.
Clients are only connected to their home peers.
Peers control the validity of client's behavior.
Hence, only a peer can attempt a double-spend.
Using the [entanglement technique](crypto.md), peer misbehavior can be cryptographically proven, and it *becomes* widely known and proven by the regular message propagation process.
[Lying peer eviction process](peerage.md) is left out of the hot path of the protocol.
The only way of peer punishment is [severing peering links (eviction)](peerage.md).

## Data model

By building on the very classic Lamport distributed system model and [logical clocks][lamport] (1978), _we completely remove the need for distributed consensus and voting_.
Transactions are either (a) valid or (b) ignored.
Namely,

* a coin is a Swarm object with an unique id (`/Coin#id`, see [specs](spec.md)),
* the only coin op is the handover op `.pay` that mentions the next owner's replica id and public key  e.g. `/Coin#id!time+owner.pay new_owner NEW_OWNER_PUB_KEY`
* the author [noop-signs](noop.md) a pay op with its own key,
* author's home peer must cross-sign the op with its own noop,
* invalid ops are ignored,
* the state of a coin object is the pair of replica id and public key of the coin's present owner,
* lies are remembered (the coin is marked as disputed), but every replica sticks with the version that arrived first.

We rely on "Lamport behavior" of processes to guarantee correctness.
For example, a replica can only generate totally ordered ops with monotonously increasing timestamps.
Once a client that owns a coin emits a payment op, it is no longer able to pay that coin to somebody else in another op.
The latter op will find that the owner has already changed.   
The only attack possibility is for a peer to send different ops in different directions (see the split coin section).

## Crypto

By relying on local linear orders, we defined a crypto coin replicated data type on top of Swarm.
Swarm employs no global ordering, no global ledger and no blockchain.
Instead, it uses local ledger*s*/chain*s*/logs and a partially ordered global op log.
Two linear orders are most important for us:

* a client replica's own log (its own ops, in creation order) and
* a peer's home log (its clients' ops in their arrival order).

It may seem that this approach should be more complex than linearization.
Surprisingly, it is quite the contrary.
These orders serve as a base for Merkle tree [entanglement](crypto.md) technique that cryptographically guarantees "Lamport behavior" of processes (monotonously increasing timestamps, no reordering, same log relayed to all peers).
The employed technic is similar to the one used in the [*git* SCM system][git-merkle].
Everything is cross-signed by everybody and nothing can be either edited or withheld.
Any lies are seen as a fork in a peer's home log.
The very existence of a cryptographically signed fork means a peer is either compromised or dishonest.

A compromised client can not rewrite its past ops or inject backdated ops; such a behavior is rejected by peers.
A compromised peer can not rollback its past transactions on two reasons:
* they must be signed by the client who made the payment first,
* once an op is propagated, it can not be rewritten retro-actively, as peers stick with the version that arrives first (see the split-coin section).

[git-merkle]: https://news.ycombinator.com/item?id=9436847

## Split-coin

Our data type definitions lead to split-coin scenarios.
A double-spending peer can split its coins, in a sense, by sending multiple payment ops in different directions simultaneously.
Hence, replicas end up in different states in respect to the coin's present owner.
Swarm has no global linearization and no consensus protocol, so such a situation can be arranged.

Now, every recipient can only pay with such a coin within its split-brain "domain".
Only this domain would recognize him/her as the owner.
Hence, the term "split-coin": the value of the coin is literally split into parts.
It is possible to pay with it, but only within some vicinity.
It is like you have a dollar, but you can only pay on US Western coast, because in other parts of the world it belongs to somebody else.
If you own that dollar in Sahara only, it hardly has any value at all.
Technically, a split coin can be reconciled by buying all of its splits.

The recipient's best strategy is to wait this uncertainty out.
On receiving a coin, a client ensures that - even if some split will surface later - its domain is already big enough for all practical purposes.
That is achieved by tracking the [entanglement matrix](matrix.md).
Given that op propagation is real-time, the matrix must reasonably settle within a second or so (cross-planet round trip time).
If it does not settle, then something wrong is happening (e.g. the recipient is offline).

So, Swarm offers no strict global consensus.
No branch "wins", no voting picks the truth.
If you lied, you lied.
Instead, it offers cryptographic data integrity and a practical majority consensus.
Given that lies are punished by peer eviction, it is possible to create the right balance of risks, profits and expenses to make split-coin a mere theoretic possibility.

And, as the saying goes, the best is the enemy of good enough.

## Conclusion

The described crypto-coin scheme is only formally a CRDT.
Actually, it builds a linear history of a single coin transactions based on local linear op histories (sessions' own logs and peers' home logs).
No global order or consensus is used.
The underlying constructs are quite well known and well tested: the Lamport model, Merkle trees and others.
The only (likely) novel part is the notion of split coin.  

The resulting scheme is likely infinitely more scalable than any blockchain/ledger based scheme.
Actually, the explained scheme bears more resemblance to real-world remittance tech (Western banking or Hawala) than to popular crypto-currencies.
Our sacrifice of symmetric peer-to-peerness is mostly imaginary, as any live peer-to-peer network employs super-peers anyway.

---

A historical note.
This text partially serves as a response to Gavin Andresen's email of 19 May 2011 that, in turn, comments on [the essay of 12 May 2011][delft] criticizing BitCoin.  

[delft]: http://www.ds.ewi.tudelft.nl/~victor/bitcoin.html
