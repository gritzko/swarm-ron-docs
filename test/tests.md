# Swarm protocol tests

These tests have a goal of:

* giving some basic case coverage for the protocol syntax and
* to serve as examples.

Every Swarm implementation is expected to read and write all the listed cases correctly.

## BATT format

The format is a rather simplistic one, serving the goal of blackbox testing.
It is as close to a plain text file as possible.

Syntax rules:

* comment lines start with `;`
* input lines start with `>`
* expected output lines start with `<` or nothing (this is the default)
* in case of multiple input/output streams, marks are generalized to
    * `stream_id>` for input and
    * `stream_id<` for output (base64 ids).

The test runner goes line by line.

1. input lines are sent to the program
2. output lines are expected to be read (error if not),
3. previously unseen stream ids trigger stream creation.

Normally, "streams" are TCP connections to a certain host/port, whether the tested program is running.
Non-concurrent (single stream) test cases can be run using stdin/stdout.
