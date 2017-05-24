## Ops

A [op](op.md) is an atomic unit of data change, the ["hourglass waist"][waist] of the RON formal model.
Ops are context-independent; each op specifies precisely its place, time and value.
Ops are immutable once created.
"Immutability changes [everything][pat]".
The implied guarantee is that very op is delivered to every (object's) replica.
Various [reducers](reducer.md) may have additional consistency requirements, e.g. [causal delivery][causal].

An op is a key-value pair of:
1. a [specifier](spec.md) and
2. some [atoms](atom.md).

### Key (specifier)

A "specifier" is an op's descriptive identifier.
The term itself is liberally borrowed from linguistics.
A specifier is a tuple of four [UIDs](uid.md):

1. the data type UID (typically a transcendent [reducer id](reducer.md), like `inc`, `lww` or `txt`),
2. the object's own UID (typically, a creation [timestamp](uid.md); transcendent for "global" objects),
3. the op's own UID (a globally unique event *timestamp*; the only allowed transcendent values are special cases of zero `0`, never `~` and error `~~~~~~~~~~`),
4. the location UID (either an actual UID of a past operation identifying the location or a transcendent UID serving as "key", "method name", etc; the interpretation is up to the reducer),

Strictly speaking, the op UID alone uniquely identifies an op.
The rest of a specifier describes the op in the most formal and concise way possible.
Thanks to UIDs, ops reference each other extensively: the object id itself is a reference to the object creation op, a location if often a reference to the op that created that location.

For non-transcendent UIDs, an obvious consistency rule is `type_id <= object_id <= location_id <= op_id`.
A referenced UID is always a past UID; due to properties of logical clocks, past UIDs have lesser values.

The Base64 op serialization is made with the following separators: `.type-id#object-id@op-id:location-id`

### Values (atoms)

Atoms are strings, integers, floats or references ([UIDs](uid.md)).
For the text-based serialization, constants follow very predictable conventions:
1. integers `=1`, `=-3` (any integer up to JavaScript Number.MAX_SAFE_INTEGER),
2. e-notation floats: `^3.1415`, `^1e+6` (explicitly separated from integers due to the fact that float arithmetics is non-associative),
3. UTF-8 JSON-escaped strings: `"строка\n线\t\u7ebf\n라인"`,
4. UID references `>1D4ICC-XU5eRJ`, `>1D4ICC-XU5eRJ[E{` (either one UID or an array of UIDs using the [bracket notation](compression.md) for prefix compression),
5. void values of `!` and `?` mark [header ops](frame.md) and [subscription ops](subscription.md) that typically have no value.

```
 Approx JSON - RON type mappings
 -------------------------------
 3.1415       =1 ^3.1415
 "string"     "string"
 null         >0
 {}           >object-id
 []           >array-id
 true         =1
```

### Example

When serialized, each of an op's eight [ints](int.md) is a Base64x64 string preceded by a non-base64 separator ( `.`, `#`, `@` and `:` for object type, object id, event stamp and event name respectively, `-` separates time values from origin ids).
For example, let's read this specifier:

```
.lww #1D4ICCEc-XU5eRJ0K @1D4IDvD4\ :title "Alice in Wonderland"
```

* an object of type `lww` (a flat last-write-wins key-value object)
* object id `1D4ICCEc-XU5eRJ0K`, i.e. created on Sun Jun 05 18:12:12 UTC 2016 (`1D4ICCEc`) by session `K` of the user `U5eRJ0` attached to the server `X` (assuming the `0163` [replica id scheme](replica.md))
* changed by the same replica on `1D4IDvD4`, Sun Jun 05 18:13:58 UTC 2016 (the repeated origin is abbreviated as a backreference `\`, see [compression](compression.md))
* the write targets the location (key) named `title`

This specifier syntax is well fit to be a key in BigTable-like prefix-compressed ordered key-value stores.
The metadata fits into the key, the data goes into the value.
In this context, the alphanumeric order of specifiers is meaningful.
It groups one object's operations together and it complies with the causal order within the object's scope (providing an arbitrary total order).

```
 Some RON op patterns
 ---------------------------------------------
 .type#id @till:loc=value   data change op
 .type#id @till:0 !         state frame header
 .type#id @0:from ?         subscription
 .type#id @till:from ?      patch query
 .type#id @till:from !      patch
 .type#id @0:0 !            zero state
 .type#id @0:0 >0           zero op
 .type#id @0:0 ?            new subscription
 .type#id @~:0 >0           object closed
 .type#id @~~~~~~~~~~"msg"  error
 .type#id @~~~~~~~~~~!      ruined state

```

[waist]: https://www.iab.org/wp-content/IAB-uploads/2011/03/hourglass-london-ietf.pdf
[causal]: https://en.wikipedia.org/wiki/Causal_consistency
[pat]: http://cidrdb.org/cidr2015/Papers/CIDR15_Paper16.pdf
