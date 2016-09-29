# Specs #

A "specifier" is a descriptive identifier for an [immutable op](op.md).
The term itself is liberally borrowed from linguistics.
The syntax of a specifier is well fit to be a key in BigTable-inspired key-value stores.

A specifier consists of four parts that fully describe the essence and the context of an operation:

1. object *type* identifies the [replicated data type](rdt.md),
2. object *id* identifies the object (typically, object id is a creation [stamp](stamp.md)),
3. a globally unique event *timestamp*, which is the [operation's](op.md) own identifier),
4. the event *name* for the operation.

Strictly speaking, the timestamp alone uniquely identifies an op.
The rest of a specifier describes the op in the most formal and concise way possible.
Swarm synchronizes partially ordered op logs.
Op forwarding is type-agnostic, so the log synchronization layer sees op values as opaque strings.
Hence, everything relevant to op storage/forwarding must fit into the spec.

Each of four parts is a [logical timestamp](stamp.md) consisting of two halves (value and origin).
Object id and timestamp are actual logical timestamps, in most of the cases.
Type and event name are most often [transcendent](stamp.md) constants (empty origin).
Still, in some scenarios, those are timestamps too.
And vice-versa, id or stamp can be constant.
Two common transcendent values for an op stamp are `!0` and `!~`, i.e. zero and infinity, used for "not yet" and "never" respectively.

All examples in this specification will use the [Base64x64](64x64.md) coding.
When serialized, each token is a Base64x64 stamp string preceded by a non-base64 separator ( `/`, `#`, `!` and `.` for object type, object id, event stamp and event name respectively).
For example, let's read this specifier:

    /LWWObject#1D4ICCEc+XU5eRJ0K!1D4IDvD4+XU5eRJ0K.title

* an object of type `LWWObject` (transcendent constant)
* object id `1D4ICCEc+XU5eRJ0K`, i.e. created by session `K` of the user `U5eRJ0` attached to the server `X` (assuming the `0163` [replica id scheme](replica.md)) on Sun Jun 05 18:12:12 UTC 2016
* changed by the same replica on `1D4IDvD4`, Sun Jun 05 18:13:58 UTC 2016
* affecting the field `title` (`LWWObject` is a very basic per-field Last Write Wins type, event names are field names, one field write per op)

The terms for different parts of a specifier are summarized below:

    specifier---object--type----class       LWWObject   name of an RDT class
        |          |     +------parameters  0           type (template) params
        |          +----id------birth       1D4ICCEc    object creation time
        |                +------creator     XU5eRJ0K    who created the object
        +-------event---stamp---time        1D4IDvD4    time of the event
                   |     +------origin      XU5eRJ0K    replica it happened at
                   +----name----method      title       method of the RDT class
                         +------scope       0           scope (0 for global)

Note that the alphanumeric order of specifiers is meaningful:
* it groups one object's operations together and
* it complies with the causal order within the object's scope (providing an arbitrary total order).
