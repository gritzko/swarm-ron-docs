 # Specs #

A "specifier" is a compound identifier for an immutable op.
The term itself is liberally borrowed from linguistics.
A specifier consists of four tokens:

* *type*, a replicated data type id,
* object *id* (typically, object id is a creation [stamp](stamp.md)),
* *stamp* (a globally unique timestamp, which is the [operation's](op.md) own identifier),
* the *name* of the operation.

Strictly speaking, the stamp alone uniquely identifies an op.
The rest of a specifier describes the op in the most formal and concise way possible.
Swarm synchronizes partially ordered op logs and op forwarding is type-agnostic.
The log synchronization layer sees op values as opaque strings.
Hence, everything relevant to op storage/forwarding must fit into the spec.

Each of four tokens is a [stamp](stamp.md) consisting of two halves (value and origin).
Object id and op stamp are actual logical timestamps, in most of the cases.
Type and op name are most often [transcendent](stamp.md) constants (empty origin).
Still, in some scenarios, those are timestamps too.
(Imagine a relational table imported into Swarm.
As the scheme changes, the op name may be a logical timestamp attached to a certain column, or data type may correspond to a particular version of the table.)
And vice-versa, id or stamp can be constants.
Although, there are only two transcendent values allowed for an op stamp: `!0` and `!~`, i.e. zero and infinity, used for "not yet" and "never" respectively.

All examples in this specification will use the [Base64x64](64x64.md) coding.
When serialized, each token is a Base64x64 stamp string preceded by a non-base64 separator ( `/`, `#`, `!` and `.` for the type, object id, op stamp and op name respectively).
For example, let's read this specifier:

    /Object#1D4ICCEc+XaUth1_K!1D4IDvD4+XaUth1_K.title

* an object of type `Object` (transcendent constant)
* object id `1D4ICCEc+XaUth1_K`, i.e. created by session `K` of the user `aUth1_` attached to the server `X` (assuming the 1-6-3 replica id scheme) on Sun Jun 05 18:12:12 UTC 2016
* changed by the same replica on `1D4IDvD4` Sun Jun 05 18:13:58 UTC 2016
* affecting the field `title` (`Object` is a very basic per-field Last Write Wins type)

Note that the alphanumeric order of specifiers is meaningful:
* it groups one object's operations together and
* it complies with the causal order within the object's scope (providing the arbitrary total order).
