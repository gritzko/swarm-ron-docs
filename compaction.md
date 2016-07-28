# Log compaction / snapshotting

Over the time, the op log grows.
In some cases, that may not pose a problem:

* in an exponentially growing data sets, the historical data takes a fraction of the volume (1=1/2+1/4+1/8...), so compaction may not worth the effort;
* the historical data may have some business value,
* in LevelDB/RocksDB, cold data gradually "submerges" to lower level chunks by itself.

In other cases, something has to be done about it:

* if only the most recent data is of interest,
* new peer boot-up needs to skip history-pumping,
* clients need to be initialized with state snapshots, not histories.

Log compaction is the process of purging old mutation history that does not affect the current state of the data (the "head" version stays the same, old versions may be forgotten).

## Compaction

Swarm data model can support Kafka-style (Kafkesque) last-write-wins log compaction for *some* data types.
Most importantly, the [simplest LWW `/Object` type](types/object.md) can be log-compacted at the database level.
As `/Object` op name is the field name, only the last write to the field affects the final state:

    /Object#created_by+author!longago+originA.x    value1
    /Object#created_by+author!recently+originB.x    value2

    /Object#created_by+author!thelast+originC.x    value3

Hence, we can compact all ops for old values of the same field.

## Snapshotting

Snapshotting is the general approach to log compaction that works for every [type](rdt.md).
An object's state is the result of all the previous operations being applied (i.e. the log).
A state snapshot is a serialized object state.
Thus, a state snapshot op on the log ([op](op.md) name `~`) overwrites all the past object's ops.
By recording a state snapshot, we make those past ops "garbage-collectable".  
A particular tactic of old op purging is left to implementations.
To make the fact of log compaction explicit, purged ops may be replaced with a [noop](noop.md), e.g. a causal link "100" noop.

## Client boot-up

A client's [object handshakes](client_handshake.md) is affected by state snapshotting in several ways.
If a client has no data on a certain object (a *fresh* replica), it should receive its current state snapshot.
A client may already have some state for the object (a *stale* replica).
A stale client should receive a *patch*., i.e. the tail of the object's op log that updates the client to the current version of the object incrementally.

Still, in some cases the peer may not serve a patch to update the stale replica.
For example, if the history was already purged.
Or, if a patch turns bigger than a snapshot.
The case of updating a stale replica with a snapshot is named *state descend*.
State descend breaks the op log based state continuity and potentially leads to complicated data races, unless handled properly and with extreme care.

The client must submit all of its yet-unacknowledged ops to its home peer before subscribing to any objects.
That way, the peer sorts out the things.
(Consider, a local client's op that was fed to the home peer, but the acknowledgement was not received, because the client went offline, and the op was purged in the peer's log before the client reconnected.)
Another potential hiccup is the case when a client authors a new op after submitting a subscription, but before receiving the response.
Then, the descended state will not have the new op.
Hence, the client will have a phantom state (the new op disappears), until it arrives from the peer again.
This discontinuity can be fixed by reapplying the new op to the descended state (and ignoring it later, once it arrives from the peer).

## Peer boot-up

A very special intriguing case is a new peer boot-up without full historical log replication.
Technically, that can be done by copying over a snapshot of the database.
If peers are served by different databases and/or database formats don't match, then the database must be cloned at the protocol level.

The receiving peer should stay passive and obey the [peer handshake]([peer_handshake.md]) protocol.
The sending peer may adopt the following approach:

* divide the database log into "compacted history" and "uncompacted tail" parts,
* first, send all the state snapshots from the history part,
* second, send ops from the tail.

In case the transmission is interrupted, the receiving peer will tell the stamp of the last op it received.
Hence, the sending peer must be able to restart sending snapshots from that particular position.
The receiving replica is not generally in the working condition before it switches to the tail-reading mode.
