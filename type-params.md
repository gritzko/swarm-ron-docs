# Type parameters

The origin half of the type id is used to convey type parameters.
Those can be generic or type-specific parameters.

The meaning of generic parameters may depend on database options.
The meaning of type-specific parameters is fully given in the type definition and does not depend on the database.

The general convention for type parameter use is `~aTcTTTTTT`, where

* `~` is the abnormal-value mark (no regular timestamps or replica ids start with '~'); type parameters reuse the origin part of the type id [stamp](stamp.md), hence the abnormal-value mark;
* `a` is access mode bits; by convention, 6 bits are used as `rwRWst`, where
    1. `rw` is the author-access mode (read and write bits)
    2. `RW` is "friend" access mode (read, write)
    3. `st` is "others" access mode (read, write)
* crypto integrity flags, format `HHSShs`
    1. `h` mandatory chain hashing bit
    2. `s` mandatory op signatures bit
    3. `HH` hashing algorithm (default is is SHA-256 truncated to 240 bits)
    4. `SS` signature algorithm (default, 1024 bit DSA)

For example, `/Counter+~x` means the author and friends can both read and write this [counter](types/counter.md), others can not. No crypto required.

`/Counter+~x33` means a counter can only incremented by one, every op must be DSA signed and SHA-256 [Merkle chain-hashed](merkle.md). This must be a really important counter then.
