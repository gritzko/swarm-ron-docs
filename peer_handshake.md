## Peer handshake ##

Peer handshake is for [full replicas](replica.md) that both possess the full op log of the database.
It is essentially a server to server handshake, albeit nothing prevents a client from having a full replica too.
Peer `.on` [ops](op.md) are scoped to their respective sending peers.
Peer `.on` may not obey the request-response pattern; peers may send their handshakes independently.

On receiving a peer on, replica sends back the section of the local log that the other peer missed and, once the catch-up phase is over, keeps relaying new ops as they arrive.
A peer replica may choose to suppress echoed ops (i.e. don't send ops back to the same peer it received them from).

A replica must maintain exactly the same op order when relaying ops live and when replaying them in a catch-up.
The local arrival order is linear and persistent.

The simplest example of a peer handshake is peers `X` and `Y` connecting for the first time to sync database `db`:

    > /Swarm#db!0.on+X
    < /Swarm#db!0.on+Y
    < ... op log
    > ... op log

In a repeated handshake, peers mention the [timestamp](stamp.md) of the last op received from each other in the past:

    > /Swarm#database!1CRE6l29+A34h~s1.on+X
    < /Swarm#database!1CRE6K+Agritzk004.on+Y

Although the op has the type-id of the metadata object (`/Swarm#database`), the stamp (`!1CRE6l29+A34h~s1`) may come from any op.
Note that the last stamp is not necessarily the max stamp.

### Non-replica subscribers ###

A peer handshake can be used by non-replica subscribers, i.e. various log consumers.
Such subscribers may not have a replica id.
Thus, their `.on` is scoped to `~`.
Similarly, the response `.on` is read-only (the stamp is `!~`, the "never" value).

    > /Swarm#database!1CRE6l29+Or5az.on+~
    < /Swarm#database!~.on+Y
