# Handshakes #

Each object- or db-level log sync starts with a handshake.
In that handshake, parties must introduce themselves (if necessary), describe the entity being subscribed to, their existing states, signal access modes (read/write), and so on.
The full subscription life cycle has four stages:

1. handshake,
2. exchange of patches,
3. continuous update exchange,
4. closing handshake.

Handshake is a two-phase request-response exchange of `.on` [pseudo-ops](op.md).
The initiator of the handshake is also denoted as a *downstream*, the receiver as an *upstream*, message propagation directions are named accordingly (downstream is from the upstream to the downstream).
Handshake's two `.on` messages are named "initial" and "return" `.on`.
A typical per-object initial `.on` [spec](spec.md) looks like:

    > /Object#1CQQ3+XaUth1_K!1CQQ3+XaUth1_K.on+Xagn0071xQ

It contains

* the type `/Object` and the id `#1CQQ3+XaUth1_K` of the object being subscribed to;
* [pseudo-op](op.md) `.on` scoped to the subscribing replica `Xagn0071xQ` (a scoped op is not relayed globally);
* version stamp `!1CQQ3+XaUth1_K` of the recentmost op/state of that object received *from the same peer* before (`!0` if none).

Note that Swarm op log is partially ordered.
Hence, in the general case, a position in the log can only be described by a version vector.
Swarm avoids VV use by relying on local de-facto arrival orders, which are linear.
A position in a particular local log can be described by a single [logical timestamp](stamp.md).

An active subscription is terminated by an `.off` operation
having essentially the same attributes.

The [value](op.md#value) of the `.on` op is unspecified; it may contain e.g. access credentials.
In the case of `.off`, the value is understood as an error message.

Three distinct types of handshakes are:

* [peer handshake](peer_handshake.md) to subscribe to the entire database log,
* [client handshake](client_handshake.md) to make a partial-dataset client subscription, and
* [object handshake](object_handshake.md) to subscribe a client to a particular object.

Database-level and object-level handshakes have the same syntax.
Database subscriptions are made by subscribing to the metadata object of the database.
That minimizes the overhead and the number of primitives.
