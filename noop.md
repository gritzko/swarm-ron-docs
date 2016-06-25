# The mighty noop

A no-op operation carries no state mutations by definition.
Still, it can convey causality information and provide [integrity metadata](crypto.md) (hashes, signatures).
A noop operation name is `.0`.
A noop MAY its own unique stamp, like a regular op.
Hence, noop affects object versions and version vectors.
Still, it does not affect the state.
A noop value consists of four optional parts, space-separated:

* a comment (square-bracketed arbitrary [Base64](64x64.md)),
* reference (a [stamp](stamp.md) of another op)
* a hash (truncated SHA-256, 40 chars)
* a signature (DSA, 54 chars)

There are eight possible combinations of the three non-comment parts, hence 8 noop patterns:

* 000 empty noop,
* 001 a signature for an implicit [rolling hash](crypto.md),
* 010 an explicit rolling hash,
* 011 a signature for an explicit rolling hash,
* 100 causal link (to an operation from another sequence),
* 101 implicit *entanglement*: a causal link and a signature for the joint hash,
* 110 causal link and an explicit joint hash,
* 111 explicit *entanglement*: signed explicit joint hash.

If a rolling hash is "implicit", the recipient is supposed to have it already, so it is not mentioned.
A *referenced hash* is a rolling hash from another sequence; a noop references it by its stamp (patterns 100-111).
A joint hash is a hash of a 80-char concatenation of:

1. the rolling hash of our sequence and
2. the referenced hash of another sequence.

For example, in a 101 pattern noop, we reference an op from another sequence, then we sign the joint hash.
It is generally recommended to reference noops, not just regular ops.
That way, we are talking about an explicitly mentioned hash.
Otherwise, the referenced rolling hash will have to be derived.

1x1 pattern noops essentially turn a Swarm op log into a cross-signed [Merkle tree][merkle].
In such a tree, eventually, all peers may cross-sign all the data.
On top of entangling noops, we may build advanced constructs, such as [entanglement matrix](matrix.md).

Some fine details.
When we reference a noop, we reference the hash *mentioned* in that op, not the rolling hash of the noop itself.
When we reference any other op, we reference the rolling hash *calculated* from that op.
If the referenced noop has an implicit hash (it is not mentioned), we must take the rolling hash from the immediate preceding op of that sequence.

A 0xx pattern noop MAY share the stamp with the preceding op, in case both the op and the noop are created by the same replica.
That way, the noop will not change version ids or version vectors.
In such a case, both definitions for the referenced hash match, as the noop mentions the rolling hash of the preceding op.

[merkle]: https://en.wikipedia.org/wiki/Merkle_tree
