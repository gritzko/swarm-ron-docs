## Client handshake ##

A client connection is opened with a meta object handshake that also counts as a client handshake.
It does not subscribe the client to any objects but the meta object.
Other objects have to be subscribed to with separate [per-object handshakes](object_handshake.md).

Note that a per-object handshake works one way only: the client subscribes to the object's updates.
Any client-side [ops](op.md) are sent upstream unconditionally and in order.
On reconnection, any unacknowledged ops are sent again, in order.
The server may acknowledge client ops in two ways:

* by echoing them back to the client or
* by sending the *last received* op stamp in the return `.on` of the client handshake.

For read-only clients, the return `.on` has a [timestamp](stamp.md) of `!~` (never).
Rejected handshakes are responded with an `.off`.
Note that the *last* received op stamp may not be the *max* received stamp if client trees are enabled (i.e. a client replica may fork off secondary client replicas with their own clocks).

Both the initial upstream `.on` op and the return downstream `.on` are scoped to the client.
In case the client did not receive a replica id yet, the initial `.on` may have an incomplete value (e.g. user bits set, peer and session bits empty).
Then, the return `.on` must be scoped to the newly granted complete replica id.

An example of a successful handshake by a new client receiving a replica id and database metadata:

    > /Swarm#database!0.on+0agn007
        Password: test123
    < /Swarm#database!1CQAneD1+X~.~
        !1CQAneD+X~.Access OwnWriteAllRead
        !1CQAneD1+X~.ReplicaIdScheme 163
    < /Swarm#database!1CQZ38+X~.on+Xagn0071xQ
        Secret: 3ff470571abe2c93cadcd43362ed7756c9fc9ac0

Note that the value format of `.on` pseudo-ops is not covered by this specification.
Values are written this way for example purposes only.
