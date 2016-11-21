# Swarm stamps / ids #

Swarm stamps are [logical timestamps][mslamp] that identify events and objects, convey event order, and causality to some degree.
Swarm stamps are pairs of [64-bit numbers](64x64.md):

1. a variable-precision timestamp and
2. a replica id.

The general idea here is that a logical timestamp can be an identifier to *anything*.
Given that each stamp is a 128-bit number, its numbering capacity is equal to an UUID.
Actually, [RFC4122 Version 1 UUID][uuid] *is* a 128-bit logical timestamp.
Swarm timestamps are extensively used in the protocol, so the format is made as compact as possible, while preserving human readability.

Stamps are serialized in [Base64x64](64x64.md) using either `+` or `-` as a pair separator, e.g. `1CQKneD1-X` (time `1CQKneD1`, [replica id](replica.md) `X`).
Replica id may be empty (numerically, 0) for *transcendent* values.
Those are global constants that are precisely defined mathematically, hence independent of any origin or time.

Swarm timestamps are based on the Gregorian calendar and not milliseconds-since-epoch because they are [hybrid][hybrid] (logical, calendar-aware).
Intuitive understanding of timestamps is a higher priority than easy calculation of time intervals.

In Base64x64, variable-precision timestamps have the `MMDHmSssnn` format.
Ten Base64 chars encode months-since-epoch, days, hours, minutes, seconds, milliseconds and an additional sequence number.
The resulting resolution is about 4 million timestamps per second.
That is often excessive, so it is OK to use shorter timestamps.
For example, `1CQKneD` (7 chars) or `1CQKn` (5 chars, `MMDHm`, no seconds - Fri May 27 20:50:00 UTC 2016).
Swarm epoch date is 1 Jan 2010 00:00:00 UTC (Unix epoch plus 1262304000000 ms).

[Replica ids](replica.md) are hierarchical, typically having three parts: peer replica (server) id bits, user id bits and session id bits.
For example, in the [0172 scheme](replica.md), replica id `Xgritzko5` has server id `X`, user id `gritzko` and session id `50`.

By convention, no regular timestamps or replica ids start with '~'.
Such values are considered *abnormal* and not treated as regular timestamps.
For example:
* `~state` is the name for the op carrying the state snapshot,
* `~` is the timestamp for "never",
*  a negative-acknowledgement op ("I will never read from you") has a [specifier](spec.md) of `@~:~on`,
* `~~~~~~~~~~` is simply "error value",
* [type parameters](type-params.md) are abnormal origin values,
* etc etc.

Philosophically, Swarm event/entity identifiers are based on a product of two very basic very classic models:

1. Lamport timestamps and
2. process trees.

It is like sequential processes (replicas) exchanging messages asynchronously AND those processes can fork off child replicas.

[lamport]: https://en.wikipedia.org/wiki/Lamport_timestamps
[hybrid]: https://www.cse.buffalo.edu/tech-reports/2014-04.pdf
[mslamp]: http://research.microsoft.com/en-us/um/people/lamport/pubs/time-clocks.pdf
[uuid]: https://tools.ietf.org/html/rfc4122#section-4.2
