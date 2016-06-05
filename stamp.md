# Swarm stamps #

Swarm stamps are [logical timestamps][mslamp] that identify events and objects, convey event order, and causality to some degree.

Swarm stamps are pairs of 64-bit numbers:

* a variable-precision timestamp and
* a replica id.

Stamps are serialized to [Base64x64](64x64.md) using `+` as a separator, e.g. `1CQAneD1+X~` (time `1CQAneD1`, replica id `X~`).
Replica id may be empty (numerically, 0) for transcendent values.
Those are global constants that are precisely defined mathematically, hence independent of any origin or time. 

Swarm timestamps are based on the Gregorian calendar and not milliseconds-since-epoch because they are [hybrid][hybrid] (logical, calendar-aware).
Intuitive understanding of timestamps is more important for us than easy calculation of time intervals.

In Base64x64, variable-precision timestamps have the `MMDHmSssnn` format.
Ten Base64 chars encode months-since-epoch, days, hours, minutes, seconds, milliseconds and an additional sequence number.
The resulting resolution is ~4mln timestamps per second.
That is often excessive, so it is OK to use shorter timestamps.
For example, `1CQAneD` (7 chars) or `1CQAn` (5 chars, `MMDHm`, no seconds - Fri May 27 20:50:00 UTC 2016).
Swarm epoch date is 1 Jan 2010 00:00:00 UTC (Unix epoch plus 1262304000000 ms).

Replica ids are hierarchical, typically having three parts: peer replica (server) id bits, user id bits and session id bits.
For example, in the 1-6-3 scheme, replica id `Xgritzk0_D` has server id `X`, user id `gritzk` and session id `0_D`.
The replica id scheme is mentioned in the database meta object (`163`, `262` etc).
In some schemes, a client can fork off its own secondary client replicas.

Philosophically, Swarm event/entity identifiers are based on a product of two very basic models: Lamport timestamps and process trees.
It is like sequential processes (replicas) exchanging messages asynchronously AND those processes can fork off child replicas.

[lamport]: https://en.wikipedia.org/wiki/Lamport_timestamps
[hybrid]: https://www.cse.buffalo.edu/tech-reports/2014-04.pdf
[mslamp]: http://research.microsoft.com/en-us/um/people/lamport/pubs/time-clocks.pdf
