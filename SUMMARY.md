# Table of Contents

* [Table of Contents](SUMMARY.md) - this document
* [Introduction](README.md) - what is the Swarm protocol
* Data replication model
    * [Replicas](replica.md) - database replicas, full and partial
    * [Order](order.md) - op order (partial, local linear)
* Protocol primitives
    * [Base64x64 numbers](64x64.md) - our sacred serialization format
    * [Stamps](stamp.md) - event/object ids for a distributed system
    * [Specifiers](spec.md) - compound event... descriptors
    * [Operations](op.md) - immutable ops are Swarm's blood cells
* [Handshakes](handshake.md) - how sync sessions start and end
    * [Peer-to-peer handshakes](peer_handshake.md) - for full database replicas
    * [Client handshakes](client_handshake.md) - for clients, to connect to a database
    * [Object handshakes](object_handshake.md) - for clients, to fetch an object and subscribe to updates
* [Replicated data types](rdt.md) - everything that can run on top of Swarm
    * [CRDT](crdt.md) - our beloved and precious Conflict-free replicated data types
    * [Swarm coin](coin.md) - an example of a non-CRDT RDT with expicit
      inconsistencies, a crypto currency
* [Crypto](crypto.md) - Swarm data integrity machinery
    * [Causality/integrity tracking](noop.md) - all-powerful noops
    * [Entanglement matrix](matrix.md) - like blockchain, but uses
      distributed math (not linear)
