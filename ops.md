# Op collection (ops)

In many cases, we have to pass around a collection of ops relating to the same object.
That is the case with RDT state snapshots, sync deltas (patches) and so on.
For that purpose, we compress a sequence of ops into a single JSON atom to pass it around in a single "container" op.
The object [id](id.md) and the type are mentioned in the container [op](op.md), so we only have to encode op ids, location ids and values.
That information is likely to have lots of redundancy, so we optimize the format to use all the benefits of [id array compression](ids.md).
Hence, an op collection is serialized as a JSON object of four fields:

1. `oid` : op ids (timestamps)
2. `loc` : op locations
3. `ref` : references
4. `val` : JSON op values

Fields 1-3 are [id arrays](ids.md), while field 4 is a JSON array of op values.
In case an op value is a reference, it goes to `ref`, otherwise to `val`.
Empty places are filled with zeros, in both cases.
Tailing zeros may be skipped, an array of zeros may not be mentioned at all.
Here is an example of three ops being converted into a container op (in this case, a state snapshot):

    #1CQKng-Rgritzko4.json@1CQKng-Rgritzko4:Field1="string"
    #1CQKng-Rgritzko4.json@1CQKnk-Rgritzko4:Field2="strong"
    #1CQKng-Rgritzko4.json@1CQKnz-Rgritzko4:Reference=#1CQKng-Rgritzko4

    #1CQKng-Rgritzko4.json@1CQKnz-Rgritzko4:~state={
        oid: "@1CQKng-Rgritzko4,kz",
        loc: ":Field1,2:Reference",
        ref: "#0;2#1CQKng-Rgritzko4",
        val: ["string", "strong"]
    }

Various replicated data types can further extend this format in a backward-compatible way.
