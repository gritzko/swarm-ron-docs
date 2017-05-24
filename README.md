# Swarm Replicated Object Notation 2.0.0 #
[*see on GitBooks: PDF, ebook, etc*](https://gritzko.gitbooks.io/swarm-the-protocol)

Swarm Replicated Object Notation is a distributed data format
designed to synchronize massively replicated datasets.
It is a text-based data format, much like XML or JSON.
While XML and JSON focus on serializing lumps of state, RON serializes a stream of changes (operations, "ops").
Even an object's state is seen as a batch of compacted ops (a "frame").

RON is [log-structured][log]: it sees data as a stream of changes (think [Kafka][kafka]).
RON is [information-centric][icn]: the data is independent of its place of storage (think [git][git]).
RON is CRDT-friendly; [Conflict-free Replicated Data Types][crdt] enable real-time data sync (think Google Docs).

Consider JSON. It expresses relations by element positioning:
`{ "foo": {"bar": 1} }` (inside foo, bar equals 1).
RON may express that state as:
```
.lww#time1-userA@\\ :bar=1
#root@time2-userB   :foo>time1-userA
```
Those are two RON ops.
First, some object has a field "bar" set to 1.
Second, another object has a field "foo" set to the first object.
RON ops are self-contained and context-independent.
Each change is versioned and attributed (e.g. at time `time2`, `userB` set `foo` to `time1-userA`).

With RON, every #object, @version, :location or .type has its own explicit [UID](uid.md), so it can be referenced later unambiguously.
That way, RON can join pieces of data correctly.
Suppose, in the above example, `bar` was changed to `2`.
There is no way to convey that in JSON, short of serializing the entire new state.
Once your dataset is bigger than HTTP headers, incremental RON updates may turn more efficient: `.lww#time1-userA@time3\ :bar=2`.
UIDs enable complex data graph structures (not just simple nesting), incremental data updates, unlimited caching, conflict resolution and other RON superpowers.
One may say, what RON metadata solves is naming things and cache invalidation.

RON makes no strong assumptions about consistency guarantees: linearized, causal-order or gossip environments are all fine.
Once all the object's ops are propagated to all the object's replicas, replicas converge to the same state.
RON formal model makes this process correct.
RON wire format makes this process efficient.


## Formal model

Swarm RON formal model has four key components:

1. an [op](op.md) is an atomic unit of data change
    * ops are context-independent; an op specifies precisely its place, time and value
    * ops are immutable once created
    * ops assume [causally consistent][causal] delivery
    * an op is a tuple of four [UIDs](uid.md) and one or more constants ([atoms](op.md)):
        1. the data type UID,
        2. the object's UID,
        3. the location UID,
        4. the op's own UID,
        5. constants are strings, integers, floats or references ([UIDs](uid.md)).
2. a [frame](frame.md) is a batch of ops
    * an object's state is a frame
    * a "patch" (aka "delta", "diff") is also a frame
    * in general, data is seen as a [partially ordered][po] log of frames
3. a [reducer](reducer.md) is a RON term for a "data type"
    * a [reducer][re] is a pure function: `f(state_frame, change_frame) -> new_state_frame`
    * reducers define how object state is changed by new ops
    * reducers are:
        1. associative,
        2. commutative for concurrent ops,
        3. optionally, idempotent.
4. a [mapper](mapper.md) translates a replicated object's inner state into other formats
    * mappers turn RON objects into JSON or XML documents, C++, JavaScript or other objects
    * mappers are one-way: RON metadata may be lost in conversion
    * mappers can be pipelined, e.g. one can build a full RON->JSON->HTML [MVC][mvc] app using just mappers.

RON implies causal consistency by default.
Although, nothing prevents it from running in a linearized [ACIDic][peterb] or gossip environment.
That only relaxes (or restricts) the choice of reducers.

## Wire format

Design goals for the RON wire format is to be reasonably readable and reasonably compact.
No less human-readable than regular expressions.
No less compact than (say) three times plain JSON
(and at least three times more compact than JSON with comparable amounts of metadata).

The syntax outline:

1. constants follow very predictable conventions:
    * integers `1`
    * e-notation floats: `3.1415`, `1e+6`
    * UTF-8 JSON-escaped strings: `"строка\n线\t\u7ebf\n라인"`
    * UID references `1D4ICC-XU5eRJ`
2. UIDs use a compact custom serialization
    * RON UIDs mostly correspond to v1 UUIDs (128 bit, globally unique, contains a timestamp and a process id)
    * RON UIDs are Base64 to save space (compare [RFC4122][rfc4122] `123e4567-e89b-12d3-a456-426655440000` and RON `1D4ICC-XU5eRJ`)
    * also, RON UIDs may vary in precision, like floats (no need to mention nanoseconds everywhere)
3. serialized ops use some punctuation, e.g. `.lww #1D4ICC-XU5eRJ :keyA @1D4ICC2-XU5eRJ "valueA"`
    * `.` starts a data type UID
    * `#` starts an object UID
    * `:` starts a location UID
    * `@` starts an op's own UID
    * `=` starts an integer
    * `"` starts and ends a string
    * `^` starts a float (e-notation)
    * `>` starts a reference (UID)
4. frame format employs cross-columnar compression
    * repeated UIDs can be skipped altogether ("same as in the last op")
    * RON abbreviates similar UIDs using [prefix compression](compression.md), e.g. `@1D4ICCE-XU5eRJ` gets compressed to `@\{E\` if preceded by `#1D4ICC-XU5eRJ`

Consider a JSON object `{"keyA":"valueA", "keyB":"valueB"}`.
A RON frame for that object will have three ops: one header op and two key-value ops.
If compressed, that frame may look like
`.lww#1D4ICC-XU5eRJ@\{E\! :keyA"valueA" @{1:keyB"valueB"` -- just a bit more than the size of the bare JSON.
That is impressive given the amount of metadata (and you can't replicate data correctly without the metadata).
The frame takes less space than *two* [RFC4122 UUIDs][rfc4122]; but it contains *twelve* UIDs (6 distinct) and also the data.


## The math

Swarm RON employs a variety of well-studied computer science models.
The general flow of RON data synchronization follows the state machine replication model.
Offline writability, real-time sync and conflict resolution are all possible thanks to [Commutative Replicated Data Types][crdt] and [partially ordered][po] op logs.
UIDs are essentially [Lamport logical timestamps][lamport], although they borrow a lot from RFC4122 UUIDs.
RON wire format is a [regular language][regular].
That makes it (formally) simpler than either JSON or XML.


The core contribution of the RON format is *practicality*.
RON arranges primitives in a way to make metadata overhead acceptable.
Metadata was a known hurdle in CRDT-based solutions, as compared to e.g. [OT-family][ot] algorithms.
Small overhead enables such real-time apps as collaborative text editors where one op is one keystroke.
Hopefully, it will enable some yet-unknown applications as well.

Use Swarm RON!


## History

* 2012-2013: started as a part of the Yandex Live Letters project
* 2014 Feb: becomes a separate project
* 2014 October: version 0.3 is demoed (per-object logs and version vectors, not really scalable)
* 2015: version 0.4 is scrapped, the math is changed to avoid any version vector use
* 2016 Feb: version 1.0 stabilizes (no v.vectors, new asymmetric client protocol)
* 2016 May: version 1.1 gets peer-to-peer (server-to-server) sync
* 2016 June: version 1.2 gets crypto (Merkle, entanglement)
* 2016 October: functional generalizations (map/reduce)
* 2016 December: cross-columnar compression
* 2017 May: Swarm RON 2.0.0

[2sided]: http://lexicon.ft.com/Term?term=two_sided-markets
[super]: http://ilpubs.stanford.edu:8090/594/1/2003-33.pdf
[opbased]: http://haslab.uminho.pt/sites/default/files/ashoker/files/opbaseddais14.pdf
[cap]: https://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed
[swarm]: https://gritzko.gitbooks.io/swarm-the-protocol/content/
[po]: https://en.wikipedia.org/wiki/Partially_ordered_set#Formal_definition
[crdt]: https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type
[icn]: http://www.networkworld.com/article/3060243/internet/demystifying-the-information-centric-network.html
[kafka]: http://kafka.apache.org
[git]: https://git-scm.com
[log]: http://blog.notdot.net/2009/12/Damn-Cool-Algorithms-Log-structured-storage
[re]: https://blogs.msdn.microsoft.com/csliu/2009/11/10/mapreduce-in-functional-programming-parallel-processing-perspectives/
[rfc4122]: https://tools.ietf.org/html/rfc4122
[causal]: https://en.wikipedia.org/wiki/Causal_consistency
[UID]: https://en.wikipedia.org/wiki/Universally_unique_identifier
[peterb]: https://martin.kleppmann.com/2014/11/isolation-levels.png
[regular]: https://en.wikipedia.org/wiki/Regular_language
[mvc]: https://en.wikipedia.org/wiki/Model–view–controller
[ot]: https://en.wikipedia.org/wiki/Operational_transformation
[lamport]: http://lamport.azurewebsites.net/pubs/time-clocks.pdf
