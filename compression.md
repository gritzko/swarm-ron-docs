## Compression

Swarm RON makes extensive use of [UIDs](uid.md): every [op](op.md) has four.
That creates the need to compress that metadata to limit the overhead.
RON UID compression relies on the fact that nearby UIDs tend to share a lot of bits.
RON employs a mix of two classic methods: prefix compression and columnar compression.

Namely, [ints](int.md) are abbreviated as deltas to some previous ("default") ints.
In the context of an op key part, UID ints default to the same UID ints of the previous op, e.g. object id origin to the previous object id origin (thus "columnar").
In the context of an op value part, UID ints default to respective ints of the previous UID.

A Base64 RON serializer may use any of the following compression tricks:

1. VARLEN - skip tailing zeroes in every [int](int.md), e.g. `time123000-origin0000` becomes `time123-origin`,
2. SKIPTOK (op key only) - entirely skip tokens if they are equal to defaults, e.g. in an object state frame only the first op has type/object ids: `.lww#1D4ICC-XU5eRJ@1D4ICCE-XU5eRJ! :keyA"valueA" @{1:keyB"valueB"`
3. PREFIX - replace common prefixes by "bracket" symbols (these are `([{}])` for 4,5,6,7,8,9 chars respectively), e.g. `time1-orig time2-orig` becomes `time1-orig(2(`
4. REDEFAULT (op key only) - use a different UUID as the default value for prefix compression, e.g. `.lww#1D4ICC-XU5eRJ@\{E\!` uses the object id as the default for the op id (here `\` means "left" and `/` means "right" redefault; a right redefault will point to the location id in the previous op instead)
5. SKIPQUANT - skip unnecessary separators, e.g. `.lww#1D4ICC-XU5eRJ\{E\!` skips `@` as the UID after the object id should be the op id anyway.
