# CRDT - Replicated Data Types

Wikipedia has a generally good [introductory article][wiki] on CRDTs.
Conflict-free replicated data types are subdivided into two main subtypes: op-based C**m**RDT and state-based C**v**RDT.
CvRDT was originally defined in terms of merging state snapshots.
CmRDT was defined in terms of applying atomic ops to the state.

Although the two types are [formally equivalent][eq], there is a great difference from the engineering perspective.
The op-based subtype needs a "causal broadcast" layer that provides *exactly-once* delivery guarantees for mutations (ops).
The state-based subtype needs more of per-object metadata to merge any two versions of that object.

Despite the differences, there was a great deal of convergence between those two types.

[wiki]: https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type
[eq]: http://link.springer.com/chapter/10.1007%2F978-3-642-24550-3_29

## [CmRDT](#CmRDT) (op-based)

The top difficulty of the op-based approach is the exactly-once op delivery.
We believe that difficulty to be greatly exaggerated.
From the standpoint of some abstract lossy message-passing environment that may be hard indeed.
Practically, many of those problems are reliably solved in existing systems.
The Swarm solves exactly-once delivery by combining:

* unique [op identifiers](spec.md),
* a log-structured storage engine and
* a TCP-like reliable transport.

Technically, the key advantage of CmRDT is the efficient propagation of mutations.
One [op](op.md) only carries its change, and not the rest of the object's state.
In line with the classic [state machine replication][smr] model, CmRDT may compress its op log into a state snapshot.

The biggest win is architectural: orthogonality of concerns.
Ops are immutable.
The broadcast layer can store and forward ops based on generic type-independent rules.
As the broadcast solves delivery issues, data type implementation is greatly simplified too.

[smr]: https://en.wikipedia.org/wiki/State_machine_replication

## [CvRDT](#CvRDT) (state-based)

CvRDTs are rather forgiving to the delivery order.
Actually, the mutation delivery order does not matter at all.
The cost of such a resilience is more of per-object metadata.
Objects can only be delivered as a full state snapshot, including all of the metadata.
That is especially painful for client-side use.
Just imagine that your Google Doc re-downloads itself on every your keystroke (Google Docs is op-based, but [not CRDT](ot)).

[Delta-enabled CvRDT][delta] resolve that inefficiency.
Deltas take the middle ground between state and ops.
They only carry the changed state.
The cost of efficiency is the new requirement: *at-least-once* delivery.
That puts "delta-enabled state-based" much closer to "snapshot-enabled op-based" in terms of abilities and requirements.

Some less-severe issues remain:
* delta handling is type-specific,
* deltas are mutable, and
* objects still carry metadata for state-to-state merge.

In our opinion, the main deficiency of CvRDT is also architectural.
There is no clear separation of concerns between the transport/broadcast layer and the CRDT/math layer.
They entangle.

[delta]: http://gsd.di.uminho.pt/members/cbm/ps/delta-crdt-draft16may2014.pdf
[ot]: https://en.wikipedia.org/wiki/Operational_transformation
