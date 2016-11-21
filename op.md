# Operation (op)

At its foundation, Swarm database is a partially ordered log of immutable operations.
The system`s guarantee is that very op is delivered to every replica exactly once, with no violations of causality.
Any state change is an op, an object`s state is created by a sequence of ops, a state delta is a bunch of ops too.
Ops are the key primitive.

An op is a key-value pair consisting of:

1. key that fully describes all the generic features of an op in the most formal way possible,
2. value (either a JSON atom or a reference).

So, everything Swarm-specific fits into the specifier, while everything specific to a data type goes into the value.
Such spec-value pairs are conveniently stored in any ordered key-value database.

A specifier consists of four tokens, where each token is an [id](stamp.md) (i.e. a logical timestamp).
Those four two-component ids are:

1. object id (a timestamp of the object creation event)
    * *birth* -- timestamp of creation
    * *creator* -- creating replica id
2. type id (typically, a predefined transcendent constant)
    * *class* (e.g. `json`, `txt`, `map`)
    * *0*, except for custom types identified by an actual timestamp
3. event id (an unique timestamp for the op itself)
    * *time* -- event timestamp
    * *origin* -- event replica id
4. location id (location within the object)
    * *method* -- (if abnormal), *name* (if transcendent)
    * *scope* -- (if abnormal)

Operations are uniquely [timestamped](stamp.md) at their origin replica using the replica's clocks.
Those timestamps are later reused to identify and reference any entity in the system: types, objects, locations within objects, etc.
Essentially, ops reference other ops.
The most common example is an object id which is normally a reference to the object creation op.
Sometimes, ids are transcendent (`txt`, `dbname`) or abnormal (`~on`, `~state`, `~gritzko`), not timestamps.

The Base64x64 serialization of an op employs four distinct separators for each kind of an id (`#`, `.`, `@` and `:`) and `=` as a value separator.
This format is line-based, one op takes one line. Two examples:

1. `#test.db@1CQC2-R:~on-Rgritzko01` -- a subscription request (`~on`) for a database (object type `db`) named `test`. The subscriber replica `Rgritzko01` already has version `1CQC2-R`, so only the diff is requested.
The value is empty.
2. `#1CQZ38-Y.json@1CQa4-Xgritzko1Q:Number=123` -- a mutation of a simple last-write-wins `json` object `1CQZ38-Y` made by replica `Xgritzko1Q` where field `Number` is set to `123`.

The value can be either a JSON atom or a reference to an object.
Examples:

1. `12345`
2. `12.34e5`
3. `"string"`
4. `"multiline\nstring"`
5. `{"json":"object"}`
6. `[1,"json",["array"]]`
7. `#1CQZ38-Y` (reference)

Complex JSON objects are still treated as "atoms", i.e. there is no possibility of addressing within them, merging changes, etc.
A complex mergeable addressable object can only be assembled out of atoms using multiple operations.

Another way of op serialization is to employ fixed width 512-bit keys made of eight 64-bit integers.
This binary form needs some special attention to big-endian vs little-endian ordering issues.

In general, immutable ops enable lots of nice-to-have features, such as time travel, debugging, caching, and suchlike.
