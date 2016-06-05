## Object handshake ##

Per-object handshakes serve to subscribe a client replica to a particular object, its current state and future updates.
The [stamp](stamp.md) of the initial `.on` pseudo-op signals the version of the object the client already has.
It is the stamp of the object's op that was either received from the server or acknowledged by the server most recently.
Special stamp values are `!0` for "nothing" and `!~` for the rare case of a write-only object.

The stamp of the return `.on` is `!~` for read-only objects or any other value otherwise.
That return stamp must be considered an acknowledgement in the case it was issued by the same client replica.

Object handshakes obey the same request-response pattern as other handshakes, with one exception.
In an object handshake, the *patch* (a catch-up op sequence) goes *before* the return `.on`.
On receiving that return `.on`, a client knows that its object state is current.

Successful no-changes subscription:

    > /Object#1CQQ3+XaUth1_K!1CQQ3+XaUth1_K.on+Xagn0071xQ
    < /Object#1CQQ3+XaUth1_K!1CQQ3+XaUth1_K.on+Xagn0071xQ

Successful update subscription:

    > /Object#1CQQ3+XaUth1_K!1CQQ3+XaUth1_K.on+Xagn0071xQ
    < /Object#1CQQ3+XaUth1_K!1Cs7L+XaUth1_K.Key1 Value1
    < /Object#1CQQ3+XaUth1_K!1Cs7M+XaUth1_K.Key2 Value2
    < /Object#1CQQ3+XaUth1_K!1Cs7M+XaUth1_K.on+Xagn0071xQ

On a [compressed connection](op.md), that exchange will likely look like:

    > #1CQQ3+XaUth1_K!1CQQ3+XaUth1_K.on
    < #1CQQ3+XaUth1_K!1Cs7L+XaUth1_K.Key1 Value1
    < !1Cs7M+XaUth1_K.Key2 Value2
    < .on

A read-only subscription (compressed):

    > #1CQQ3+XaUth1_K!1CQQ3+XaUth1_K.on
    < #1CQQ3+XaUth1_K!1Csu_+XaUth1_K.FieldName Some text
    < !~.on
