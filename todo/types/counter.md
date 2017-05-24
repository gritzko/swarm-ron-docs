# Counters

CmRDT counters are rather trivial.
Each operation contains a number, either positive or negative.
The resulting state is a sum of those numbers.

The only op type is `.add`.
Op or state value is a decimal number, possibly preceded with '+' or '-' for positive or negative values, e.g. `+1`.

Counter type id `/Counter`.

The only [type-specific parameter](type-params.md) is *mode*.

Counter mode switch bits are:
* 0 bit: grow-only counter (does not accept negative increments),
* 1 bit: by-one counter (the only valid increments are +1, -1)

Example:

`/Counter+~x3` is a counter type that can only be incremented and only by 1 (`3`, 3, 000011), read-write access is available to the author and "friends" (`x`, 60, 111100).
