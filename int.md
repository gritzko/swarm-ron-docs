## Base64x64 integers, timestamps and ids

Swarm RON employs lots of ids, timestamps and pre-defined constants all unified as 128-bit RON UIDs consisting of two 64-bit parts.
64-bit integers are well supported on all platforms, with the notable exception of JavaScript. JavaScript numbers are [dangerous to use][snowflake] in this context.
Hence, in human-readable serializations (URL paths, file names) and in JavaScript code RON relies on Base64-serialized integers.

Swarm RON Base64 variant satisfies two special requirements.
First, RON has to do lots of causality-and-order comparisons with UIDs/timestamps.
Hence, the natural alphanumeric ordering of Base64 strings must be the same as the original numeric order of integers.
That rules out any common variety of Base64 from being used, as [Base64][base64] symbols do not go in their ASCII order.

Second, full 64 bits is a bit too much sometimes, so a variable-length encoding is a must (see Google's [varints][varint]).

Hence, RON employs the following Base64x64 encoding:

* 64-bit numbers are represented as ten Base64 chars (10x6=60, 4 higher bits are reserved),
* our Base64 variety is `0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ_abcdefghijklmnopqrstuvwxyz~` (non-standard),
* *tailing* zeroes are skipped, e.g. `1230000000` is shortened to `123`.

The tail-skip rule is a bit counter-intuitive, as the normal Arabic notation skips *leading* zeroes, e.g. Arabic 0000123 is 123.
But this way, the numeric order of `uint64_t` matches the alphanumeric order of our Base64, which is not the case with Arabic numbers.

So, Base64x64 is like [varint][varint], but for text-based formats and it preserves the order.  It is mostly used for [UIDs](uid.md), timestamps and identifiers.

[varint]: https://developers.google.com/protocol-buffers/docs/encoding#varints
[snowflake]: https://dev.twitter.com/overview/api/twitter-ids-json-and-snowflake
[hybrid]: https://www.cse.buffalo.edu/tech-reports/2014-04.pdf
[base64]: https://tools.ietf.org/html/rfc4648#page-5
