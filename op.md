# Operation (op)

At its foundation, Swarm database is a partially ordered log of immutable operations.
The system's guarantee is that very op is delivered to every replica exactly once, with no violations of causality.
Any state change is an op, an object's state is created by a sequence of ops, a state delta is a bunch of ops too.
An op is the key primitive.

An op is a key-value pair:

1. the key fully describes all the generic features of an op in the most formal way possible,
2. the value is either a JSON atom or a reference.

So, everything Swarm-specific fits into the key, while everything specific to a data type goes into the value.
Such spec-value pairs are conveniently stored in any ordered key-value database.

The key consists of four [ids](id.md), and each id consists of two [Base64x64](64x64.md) numbers (most often, an id is a logical timestamp).
Those four-by-two numbers are:

1. object id (typically, the timestamp of the object creation event)
2. type id (normally, a predefined transcendent constant)
3. event id (always a timestamp)
4. location id (location within the object, most often some past event id)

Operations are uniquely [timestamped](id.md) at their origin replica using the replica's clocks.
Those timestamps are later reused to identify and reference any entity in the system: types, objects, locations within objects, etc.
Essentially, ops reference other ops.
The most common example is an object id which is normally a reference to the object creation op.
Sometimes, ids are transcendent (`txt`, `dbname`) or abnormal (`~on`, `~state`, `~gritzko`), not timestamps.

The [Base64x64](64x64.md) serialization of an op employs four distinct separators for each kind of an id (`#`, `.`, `@` and `:`) and `=` as a value separator.
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
That binary serialization needs some special attention to ordering related issues (big-endian vs little-endian).

In general, immutable ops enable lots of nice-to-have features, such as time travel, debugging, caching, and suchlike.
The must-have feature of data synchronization mostly boils down to delivering every op to every replica.

<table>
    <thead>
    <tr><th colspan='9'><center>Table: key parts and their meanings
        for timestamp ids, abnormal and transcendent ids</th></tr>
    <tr>
        <td><i><b></td>
        <td colspan='2'><b>object id</td>
        <td colspan='2'><b>type id</td>
        <td colspan='2'><b>event id</td>
        <td colspan='2'><b>location id</td>
    </tr> </thead>
    <tr>
        <td><i>timestamp id</td>
        <td>object birth time</td>
        <td>object author replica</td>
        <td colspan='2'>custom type id</td>
        <td>event time</td>
        <td>event origin</td>
        <td colspan='2'>past event id</td>
    </tr>
    <tr>
        <td><i>abnormal id</td>
        <td>local object name (abn)</td>
        <td>object scope</td>
        <td>type name</td>
        <td>type parameters (abn)</td>
        <td colspan='2'>never used</td>
        <td>op name (abn)</td>
        <td>op scope</td>
    </tr>
    <tr>
        <td><i>transcendent id</td>
        <td>global object name</td>
        <td>0</td>
        <td>type name</td>
        <td>0</td>
        <td>0 or ~</td>
        <td>0</td>
        <td>0 or method name</td>
        <td>0</td>
    </tr>
</table>
