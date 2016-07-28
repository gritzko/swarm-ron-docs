# `Object` RDT

`Object` is a basic last-write-wins data type.
An object has named *fields* which can be set to arbitrary JSON values.
One op sets one field.
Out of two edits of the same field, the one with greater timestamp wins.
Hence, `Object` does field-by-field last-write-wins merge.
To simplify op format and processing, the op name matches the field name.
Hence, object fields have [Base64](64x64.md) names up to 10 characters.
Reserved op names can not be used as Object field names (`on`, `off`, `~`, `0`, `error`).
Tilde can not be used in field names.
There are no other restrictions on field name vocabulary.

A serialized `Object` op looks like this:

    object.put(100, 200, "value")

turns into

    /Object#TimeCreatd+author!TimeModifd+origin.fieldName    "value"

In addition to field name based access, `Object` also supports array-like access by integer indexes, either one- or two-dimensional.
Indexed values are serialized using regular Base64 encoding (leading zeroes skipped) and `~` as an op name prefix.

    matrix_object[1][65] = "cell"

turns into

    /Object#TimeCreatd+author!TimeModifd+origin.~1+~11    "cell"

Note that single-indexed accesses read/write a vector which does not belong to the two-indexed matrix.
Single-indexed accesses get serialized as `.~0`, `.~1`, `.~3`... while two-indexed turn into `.~0+~0`, `.~1+~0`...
Indexed access supports vectors up to 2^54 cells and matrices up to 2^54x2^54 (although it is recommended to limit yourself to 2^40 because JavaScript has no 64 bit integers).

`Object` state snapshot is serialized as *normalized* JSON map where keys are op names and values are values:

    {"fieldName":"value","~0":12345,"~1+~11":"cell"}
