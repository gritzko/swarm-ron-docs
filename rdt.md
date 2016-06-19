# Replicated data types

An RDT is a formal construct that describes anything that can work over a partially ordered op log.
It can be a [CRDT](crdt.md) (either subtype), like [Counter](types/counter.md) or anything else.
One popular example is RPC (remote procedure call) which employs scoped operations to avoid global op propagation.
Another peculiar example is a Swarm-based [crypto currency](coin.md).

General principles for every kind of RDT:

1. RDT objects have a state which is mutated by applying [ops](op.md)
2. the same set of ops produces the same state ("same" means bitwise identical)
3. reordering of concurrent ops does not affect the resulting state
4. external APIs of an RDT should be idiomatic to their language of implementation, hence they may vary from language to language

A good RDT must have:
1. a clear mission and a discussion of tradeoffs,
2. proof of correctness,
3. test cases consisting of op sequences and resulting states (preferably concurrent cases too)
4. ideally, JavaScript, C++ and Java implementations.
