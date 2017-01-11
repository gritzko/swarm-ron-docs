# Swarm timestamp/id

Swarm ids identify events, objects and other entities, convey event order and (partially) causality.
Swarm ids are [logical timestamps][mslamp].
Technically, an id is a pair of [64-bit numbers](64x64.md):

1. a variable-precision time value and
2. a replica id.

The general idea here is that such a timestamp can be an identifier to *anything*.
Given that each stamp is a 128-bit number, its numbering capacity is equal to an UUID.
Actually, [RFC4122 Version 1 UUID][uuid] *is* a 128-bit Lamport timestamp.
Swarm ids are extensively used in the protocol, so the standard UUID notation would turn fatally verbose.
Swarm id format is made as compact as possible, while preserving human readability.
Instead of hex, it employs Base64.
Also, the 128 bit capacity is most often excessive, so Swarm ids use variable length encoding.

Stamps are serialized in [Base64x64](64x64.md) using either `+` or `-` as a separator, e.g. `1CQKneD1+X` (time `1CQKneD1`, [replica id](replica.md) `X`). Plus is used for original event ids, minus for [derived](derived.md) event ids.

Swarm timestamps are are [hybrid][hybrid] (logical, calendar-aware).
They are based on the Gregorian calendar, which may seem unusual.
Normally, timestamps use milliseconds-since-epoch.
In our case, intuitive understanding of timestamps was a higher priority than easy calculation of time intervals.

In the Base64x64 encoding, Swarm id time values have the format of `MMDHmSssnn`.
Ten Base64 chars encode months-since-epoch, days, hours, minutes, seconds, milliseconds and an additional sequence number.
(Swarm epoch s 1 Jan 2010 00:00:00 UTC.)
The resulting timestamp resolution is about 4 million timestamps per second, which is often excessive.
Like with floating point numbers, Swarm timestamp precision may be adjusted as necessary.
For example, it is OK to use 7 chars `1CQKneD` or 5 chars `1CQKn` (`MMDHm`, no seconds - Fri May 27 20:50:00 UTC 2016).

The other part of a Swarm id is a replica id ("process id" in Lamport terms).
[Replica ids](replica.md) are hierarchical, typically having three parts: peer replica (server) id bits, user id bits and session id bits.
For example, in the [0172 scheme](replica.md), replica id `Xgritzko5` has server id `X`, user id `gritzko` and session id `50`.

Every Swarm timestamp is a Swarm id, but not vice-versa.
A Swarm id may be a global *transcendent* constant that is precisely defined mathematically, hence independent of any origin or time.
For transcendent ids, replica id is empty (numerically, 0).
Transcendent ids are not timestamps.

Similarly, no regular timestamps or replica ids start with `~`.
Such ids are considered *abnormal*. Abnormals are not timestamps.
For example:
* `~state-Rgritzko1` is the name for an op carrying an object's state to replica `Rgritzko1`,
* `~` is a timestamp for "never",
*  a negative-acknowledgement op ("I will *never* read from you") has `~` as its version field,
* `~~~~~~~~~~` is simply "error value",
* pseudo-operation ids are abnormal (`~on`, `~off`),
* etc etc.

Philosophically, Swarm event/entity identifiers are based on a product of two very basic very classic models:

1. Lamport logical timestamps and
2. process trees.

It is like sequential processes (replicas) exchanging messages asynchronously AND those processes can fork off child replicas.

[lamport]: https://en.wikipedia.org/wiki/Lamport_timestamps
[hybrid]: https://www.cse.buffalo.edu/tech-reports/2014-04.pdf
[mslamp]: http://research.microsoft.com/en-us/um/people/lamport/pubs/time-clocks.pdf
[uuid]: https://tools.ietf.org/html/rfc4122#section-4.2
