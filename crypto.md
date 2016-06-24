# Crypto primitives

The Swarm protocol provides rather strong delivery guarantees (exactly once, no causal reordering).
Still, the raw protocol itself does not protect against Byzantine faults and silent data corruption.
But, the very nature of immutable operations enables rich data integrity mechanics.

The range of available options spans from simple causality information to mandatory data hashing to massively redundant data cross-signing.
Swarm can be ran both as a federated database and as an [open super-peer network](peerage.md).
The likely absence of single central authority makes the data integrity toolset vital.

Swarm crypto is different from blockchain based solutions as it does not rely on proof-of-work and miners, but instead resorts to peer reputations.
In the most strict setup, a single trustworthy peer can guarantee the integrity of the entire database.
The BitCoin/Ethereum story shows that we can not avoid having reputable system custodians anyway.
So the key design question was: do reputations suffice?
May the proof-of-work part turn unnecessary?
The answer is mostly yes.

Swarm crypto model is much closer to git than to blockchain.
Swarm crypto allows any participant to sign all the visible data.
That effect compounds, because peers and clients unavoidably cross-sign each other's signatures.
Hence, the past information can not be possibly altered as long as at least one signatory is not compromised.

## Rolling hashes

A rolling hash chain guarantees that no ops have been withdrawn from a sequence or corrupted in transit.
Such a hash chain can be defined for any linear op sequence, such as:

* single replica op log (includes all ops created and stamped by the replica),
* peer op log (included all the ops by the peer's clients in their arrival order) and
* single-origin object op log (ops of a single replica over a single object).

A rolling hash is defined irrespectively of whether it is mentioned explicitly in the respective op log.


Swarm employs truncated SHA-256 hashes serialized as 40 Base64 numbers (the tailing 16 bits of SHA-256 are skipped, the default all-zero value is written as `0`).

A rolling hash at position `0` (no ops) is all zeroes (written `0`).
A rolling hash at position `stamp+X` is a SHA hash of:
* the rolling hash at the previous position (40 bytes)
* the op at position `stamp+X` in the [binary form](op.md) (8*2*4=64 bytes for the spec, 4 bytes for the size, then the value)

The binary for is chosen because it is the least ambiguous and has no variants of representation (like optional tailing zeros or alternative serializations).
A rolling hash can be added to the op log using the [noop op](noop.md), op name `.0`.
The fact of mentioning the rolling hash on the log changes the log's rolling hash.
The type of the rolling hash (replica, peer or object) can be derived from the stamp and type-id of the noop:
* peer rolling hash is stamped with the peer's timestamp (e.g. `/Swarm#database!time+XY.0` in the 0*2*5*3 [replica id scheme](replica.md)),
* replica's rolling hash is stamped with the replica's stamp (`/Swarm#database!time+XYauserSsn.0`),
* single-origin object rolling hash has the type and the id of the hashed object (`/Object#created+author!time+XYauserSsn.0`).

(Fine detail: A peer can edit and sign the [meta object](meta.md) using its [pocket session](pocket.md), hence metadata's object hashes differ from full-database log hashes by the origin of the stamp.)

Note that a rolling hash needs no full recalculation (the full history may not even be available); it is calculated incrementally, op by op.
In case an op log starts with a state snapshot, a rolling hash can be provided by a same-type, same-id, same-stamp noop immediately following the state snapshot.

As a noop has an unique timestamp, it can be referenced.
Essentially, it is a building block for higher-order constructs that allows to prove the integrity of the causal past if any event.

The need of creating and checking rolling hashes is signaled in
* [type parameters](type-params.md) for separate data types or
* in the type parameters of the database, for all types.

## Causal links

Many important op sequences are not linear, but partially ordered.
Namely, arrival orders at different replicas may vary: concurrent ops can go in
different orders.
To entangle different linear sequences into a proper causal [cone of the past][minkowski], Swarm employs causal links.
Namely, a noop can cite a rolling hash of one sequence in the other sequence, thus linking two sequences cryptographically.
It is recommended to always cite explicit hashes (i.e. those mentioned in existing noops), albeit it is possible to cite an implicit hash (not mentioned in a noop, but derived from the sequence directly).

Causal links enable:

* full-swarm full-database op log hashes (made by peers),
* client hashes (made by linking all the client's sessions) and
* object op log hashes (for the full op log of an object).

The need for causal linking is signaled in type parameters.


[minkowski]: https://en.wikipedia.org/wiki/Light_cone

## Signatures

What can be hashed can be signed too.
Noops can contain signatures put by their author replicas.
The default algorithm is DSA, 2048 bits.
Signatures are serialized as 54 Base64 chars (higher 4 bits of the first char are zeros).
Public keys for each replica are listed in the replica's meta object `/Replica#0+ReplId`.

In-transit signature checks are generally OPTIONAL.
A peer MUST check the signature if

* an op is received from the client
* the type parameters signal a mandatory signature.
