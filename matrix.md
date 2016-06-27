# Entanglement matrix

Because of [rolling hashes](crypto.md), an attacker can neither inject, nor withhold any operations from other peer's op streams.
Due to cryptographic [entanglement](noop.md), an attacker can not censor streams (i.e. selectively suppress/relay peers).
Still, there is another group of attacks, most notably the famous double-spending attack, that depend on the attacker's ability to broadcast different versions of reality to different peers.

Once the attacker sends out contradictory ops, that creates a swarm split-brain.
Unless the swarm is physically permanently separated, the contradiction will be detected as soon as both versions of reality are known to all peers.
Still, there is a window of opportunity for the duration of the split-brain.

The entanglement matrix is a cryptographic construct that ensures that all peers see the same, without any global linearization or blockchains.

As such, it limits the duration of undetected split-brains.
Ultimately, it allows to wait out the uncertainty: once enough peers accepted the op, we will see that from the matrix.
From that point on, the protocol's causality guarantees prevent any split-brain from happening (in regard to that op).

An entanglement matrix needs no other primitives but [signed noops](noop.md).

Suppose, three peers (A, B and C) each generated three signed noops at different points in time:
* A at 1, 2 and 4,
* B at 2, 3 and 5,
* C at 1, 4 and 5.

Peers are connected in a chain A-B-C.
The following causality diagram reflects both op propagation and cryptographic entanglement of those noops:

    A -1-2---4---
          \ /
    B ---2-3---5-
        /   \ /
    C -1-----4-5-

For example, A has seen 3+B, but not 5+B yet.
C has seen 2+A, as it was relayed by B just before 3+B.
We say that 3+B *entangles* 2+A and 1+C, because it signs a [Merkle tree][merkle] that has both those ops.

The entanglement matrix reflects which peer's noop the other peer has already seen (horizontal: senders, vertical: observers).

        A   B   C
    A   4   3   1
    B   2   5   4
    C   2   3   5

Essentially, an entanglement matrix is a cryptographically hardened [matrix clock][mc].
We use peer [full-swarm signed rolling hashes](crypto.md) delivered as [noops](noop.md), e.g.

    /Swarm#database!3+B.0 2+A HASH SIGNATURE

On every entangling noop, we add it to the matrix.
We also continue by adding recursively any past noops it further entangles (i.e. 3+B entangles 2+A for C and 1+C for A).

The entanglement matrix can answer various queries.
For example, we may see which peer's ops are certainly known to everyone already: 2+A, 3+B, 1+C.
We may see which ops are known to a majority of peers: 2+A, 3+B, 4+C.

Some closing remarks on scalability.

Even in a super-peer network, the number of peers may be large.
An entanglement matrix has a size of O(N^2), i.e. quadratic.
The fact that we need such a big data structure may seem depressing at first.
Practically, a regular peer or client hardly needs the full matrix.
A replica may need to know which of *its own* ops are reliably disseminated.
To answer that question, it needs one column from the matrix (and a little bit more to implement recursion, but O(N) anyway).

A client replica may need one additional step to use an entanglement matrix.
Namely, it has to link the op of interest to the nearest home peer's noop.
Practically, that is a hash chain validation.
Once a client can see that the op is covered by a home peer's noop, it may track the progress of the entanglement matrix till it shows majority acceptance of the op in question.
Note that the client does not need the full log or the full entanglement matrix.
The quorum proof can be made with a segment of the op log, in both cases.

[mc]: https://en.wikipedia.org/wiki/Matrix_clock
[merkle]: https://en.wikipedia.org/wiki/Merkle_tree
