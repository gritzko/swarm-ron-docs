## Frames

A frame is a non-empty array of [ops](op.md) (order is significant).
Op frame processing is transactional:
A frame is either processed in its entirety or not processed at all.
While [ops](op.md) are immutable, frames of ops get routinely created and rehashed.

Swarm RON frames provide:
1. lightweight transactional semantics,
2. frame-level op [compression](compression.md) that fully exploits the fact that close UIDs tend to be close numerically,
3. a great unifying abstraction:
  * an object's state is a frame;
  * [reducers](reducer.md) consume frames and produce frames,
  * a patch is a frame carrying a difference between two states,
  * an object graph snapshot is a frame too,
  * a single op travels around as a frame of one op.

Apart from immutable payload ops, a frame may have its own *header* op.
A header op consists of:
1. reducer id (the one that produced the frame),
2. object id (or 0 for omnivorous reducers),
3. end version (most often, the last reduced op id),
4. start version (actual for patches).
5. void value (`!`)
