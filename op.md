# Operations

At its base, Swarm is a partially ordered log of immutable operations.
The key guarantee is that very op is delivered to every (object's) replica exactly once, with no violations of causality.
There are three operation classes:

* pseudo-ops: (un)subscriptions and errors (`.on` `.off` `.error`)
* state snapshots (`.~`)
* state mutations (anything else)

An op is a key-value pair of

1. a [specifier](spec.md) and
2. a value.

Such spec-value pairs are conveniently stored in any ordered key-value database.
Everything Swarm-related fits into the specifier.
Everything specific to a data type goes into the value.
This specification does not cover value formats and sees values as opaque buffers.

Operations are uniquely [timestamped](stamp.md) at their origin using local logical clocks.
Hence, ops can reference each other. The most common example is the object id which is a reference to the object creation op.

Immutable ops enable lots of nice-to-have features, such as time travel, debugging, caching, and suchlike.
They also enable our #1 must-have feature, synchronization.

## Storage/transmission op format

Swarm's primary data format is a stream of operations represented as spec-value pairs.
A [spec](spec.md) is four [logical timestamps](stamp.md), technically eight 64-bit numbers.
A value is an arbitrary blob.
There are three straightforward ways to serialize ops: binary, text-based and JSON-based.

### Binary

Binary is the most straightforward (in the C++ sense) encoding.
An op is simply dumped from its in-memory form as three records:

* 64x2x4=512 bits for the spec,
* 32 bits for value length,
* value buffer

Longs and ints are little-endian, spec components go in their natural order (type, id, stamp, op name), each components goes as (value, origin).

### Text

A line-based human-readable format encodes each spec as a sequence of Base64 tokens and non-Base64 separators as described in [spec.md](spec.md).
There are four formats to encode the value:

* empty (no value, just a newline),
* single-line value (tab, value, newline),
* multiline value (newline, then indented value lines) and
* explicit length value.

Empty value:  spec LF

    /Swarm#database!0.on

Single line value:  spec HT value LF

    /Object#1CQZ38+Y~!1CQa4+Xusernm1Q.NumbrField 123

Multiline value:  spec LF ( HT line LF ) *

    /Swarm#database!1CQAneD1+X~.~
        !1CQAneD+X~.Access OwnWriteAllRead
        !1CQAneD1+X~.ReplicaIdScheme 163
    !1CQa5+Xusernm1Q.Multiline
        First line
        Second line
            Indented third line

Explicit length value:  spec = base64length LF buffer LF*

    !1CQa6+Xusernm1Q.Buffer=A
    raw buffer

### JSON

    [ [spec64x64, valueString], [spec64x64, valueString],... ]

The JSON format reuses [Base64x64](64x64.md) encoded specs.
Compared to the text-based format, it reduces the need for manual parsing.
The transport is supposed to be discrete or frame-based (i.e. HTTP or WebSocket), as there is no way to chain such fragments naturally.

## Compression

Compression is a format-neutral trick.
Every spec has exactly four components.
As subsequent op specs are more likely to share some tokens, such repeated tokens can be omitted altogether.

Technique A is to omit fully identical tokens.
For example,

```
/Object#1CQQ3+XaUth1_K!1Cs7L+XaUth1_K.Key1 Value1
/Object#1CQQ3+XaUth1_K!1Cs7M+XaUth1_K.Key2 Value2
/Object#1CQQ3+XaUth1_K!1Cs7M+XaUth1_K.on+Xagn0071xQ
```

Can be compressed to:

```
/Object#1CQQ3+XaUth1_K!1Cs7L+XaUth1_K.Key1 Value1
!1Cs7M+XaUth1_K.Key2 Value2
.on
```

Technique B is to omit repeating halves (values and origins) separately.
This necessitates an adjustment to the text-based serialization.
Transcendent values now must mention their 0 origin explicitly, e.g. `/Object+0`.

Technique A is the default behavior in case a deserializer encounters a token omission and there are no prior instructions.

In the case of binary coding, there is no way to omit tokens, but repeated tokens can be replaced with special values (e.g. 11111..11).
It is an open question whether it pays off, compared to straight gzip.

JSON based encoding benefits from shorter specs exactly the same way as the text-based encoding.
Compression does not span JSON frames.
