# Merkle hash chain

The Swarm protocol provides rather strong delivery guarantees (exactly once, no causal reordering).
Still, the raw protocol itself does not protect against Byzantine faults, bit flips, silent corruption.
To address that, a Swarm replica MAY employ crypto: Merkle hash chains and signatures.

## Hash chains

A hash chain guarantees that no ops have been withdrawn from a sequence or corrupted in transit.
All hashed ops issued by a replica form a single Merkle chain.
There is no possibility to provide finer grained (e.g. per-object) hash chains at the protocol level.
The need of hash chaining is signaled in [type parameters](type-params.md).

A hash pseudo-op is transmitted immediately after its original [op](op.md).
Its [specifier](spec.md) is exactly like the op's, except the op name is `.~hash`.
A replica is REQUIRED to check a hashed op with its hash once the op is received, before any further processing.
In case of hash mismatch, the op MUST be dropped and the connection MUST be severed.

The default hashing scheme is SHA-256.
The hash is truncated to 240 bits and serialized as 40 [Base64x64](64x64.md) chars.
The hashed content is:
* the op serialized in the [text format](op.md), empty, single-line or explicit-length, not multiline.
* the hash of the previous op, as an op in the text format, single-line, abbreviated.


## Examples.

Original op:

    /Object+~x02#1CQZ38+Y~!1CQa4+Xusernm1Q.NumbrField    123

Hashed content (there is no previous op):

    /Object+~x02#1CQZ38+Y~!1CQa4+Xusernm1Q.NumbrField    123
    !0.~hash    0

Hence, the hash is:

    dc11a469d1fd1d1c3817e928119ce97b494470185e637158b2656e34651a46e0

Then, the hash op is:

    /Object+~x02#1CQZ38+Y~!1CQa4+Xusernm1Q.~hash SmQ4eTVDT13tNEIdH9EuwKK_lH5jZNL8nbMUp6HQ

The next hashed op would probably be:

    /Object+~x02#1CQZ38+Y~!1CQa5+Xusernm1Q.NumbrField    345

its hashed content:

    /Object+~x02#1CQZ38+Y~!1CQa5+Xusernm1Q.NumbrField    345
    !1CQa4+Xusernm1Q.~hash    SmQ4eTVDT13tNEIdH9EuwKK_lH5jZNL8nbMUp6HQ

Its hash is:

    /Object+~x02#1CQZ38+Y~!1CQa5+Xusernm1Q.~hash    f_77ABq5718VLMNguOFMcW~eg2MzKVSivQ7ttF9d    

Hence, it must be sent on the wire as:

    /Object+~x02#1CQZ38+Y~!1CQa5+Xusernm1Q.NumbrField    345
    .~hash    f_77ABq5718VLMNguOFMcW~eg2MzKVSivQ7ttF9d    
