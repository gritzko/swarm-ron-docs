# Op collection (ops)

In many cases, we have to pass around a collection of ops relating to the same object.
That is the case of RDT state snapshots, sync deltas (patches) and some others.
For that purpose, we compress a sequence of ops into a single JSON atom and move that around as an op value.
A collection of ops is likely to have plenty of redundancy, so we optimize the format to use all the benefits of [id array compression](ids.md).
As object id and type are mentioned in the containing op, we only have to encode op ids, location ids and values.
Hence, an op collection is serialized as a JSON object having four fields:

1. `ids` : op ids (timestamps)
2. `loc` : op locations
3. `ref` : references
4. `val` : JSON op values

Fields 1-3 are id arrays, while field 4 is a JSON array of JSON atoms (op values).
In case op value is a reference, it goes to `ref`, otherwise to `val`.
Empty places are filled with zeros, in both cases.
Here is an example of three ops being converted into a state snapshot:

    #1CQKneg-Rgritzko4.json@1CQKng-Rgritzko4:Field1="string"
    #1CQKneg-Rgritzko4.json@1CQKnk-Rgritzko4:Field2="strong"
    #1CQKneg-Rgritzko4.json@1CQKnz-Rgritzko4:Reference=#1CQKneg-Rgritzko4

    #1CQKneg-Rgritzko4.json@1CQKnk-Rgritzko4:~state={
        ids: "@1CQKng-Rgritzko4,kz",
        loc: ":Field1,2:Reference",
        ref: "#0;2#1CQKneg-Rgritzko4",
        val: ["string", "strong", 0]
    }

This format can be further extended by replicated data types in a backward-compatible way.
