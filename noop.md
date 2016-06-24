# The mighty noop

A no-op operation carries no state mutations by definition.
Still, it can convey causality information and provide [integrity metadata](crypto.md) (hashes, signatures).
A noop operation name is `.0`.
A noop has its own stamp, like a regular op.
Hence, noop affects object versions and version vectors.
Still, it does not affect the state.
A noop value consists of four optional parts, space-separated:

* a comment (square-bracketed arbitrary [Base64](64x64.md)),
* reference (a stamp of another op)
* a hash (truncated SHA-256, 40 chars)
* a signature (DSA, 54 chars)

There are eight possible combinations of the three non-comment parts, hence 8 noop patterns:

* 000 empty noop,
* 001 a signature for an implicit [rolling hash](crypto.md),
* 010 an explicit rolling hash,
* 011 a signature for an explicit rolling hash,
* 100 causal link (to an operation from another sequence),
* 101 causal link and a signature for the joint hash,
* 110 causal link and an explicitly joint hash,
* 111 signed explicit joint hash.

If a rolling hash is "implicit", the recipient is supposed to have it already, so it is not mentioned.
A *cited hash* is a rolling hash from another sequence; a noop references it in patterns 100-111.
A joint hash is a hash of a 80-char concatenation of:

1. the rolling hash of our sequence and
2. the cited hash of another sequence.

For example, in a 101 pattern noop, we reference an op from another sequence, then we sign the joint hash.
It is generally recommended to reference noops, not just regular ops.
That way, we are talking about an explicitly mentioned hash.
Otherwise, the referenced rolling hash will have to be derived.
