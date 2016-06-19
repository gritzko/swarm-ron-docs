# Swarm protocol tests

These tests have an objective of:

* basic case coverage for the protocol syntax and
* providing an examples.

Every Swarm implementation is expected to read and write all the listed cases correctly.

## BATT format

The format is a rather simplistic one, serving the goal of blackbox testing line-based protocols.
It is as close to plain text file as possible.

Syntax rules:

* comment lines start with `;`
* input lines start with `>`
* expected output lines start with `<` or nothing (this is the default)
* in case of multiple input/output streams, marks are extended to
    * `stream_id>` for input and
    * `stream_id<` for output (base64 ids).

The test runner reads a `.batt` file line by line, so

1. input lines are sent to the tested program
2. output lines are expected to be read from the program (error if not),
3. previously unseen stream ids trigger new stream creation.

Normally, "streams" are TCP connections to a certain host/port where the tested program is running.
Single stream test cases can be run using stdin/stdout.
