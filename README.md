# multiplexing
Multiplexing should be a low level primitive of computing

This repository describes a primitive for describing multiplexations.

# API specification

Ultimately multiplexing is the volcano pattern of multiple independent virtual machines running in parallel concurrently each raising events that are then ordered by a scheduler and serialised.

```
time = new VirtualDimension();
Kernel = new Dimension()
Lightweight = new Dimension();
scheduler = new Scheduler();
inputs = new Source();
ast = new Stackframe();

```

The scheduler has to work in a stream fashion.

Stackframes act as logical groupings of data.

Stackframes are themselves scheduled.

Intermediate representation.

Parallel communicating processes is essentially multiplexing multiple instructions multiplexed over kernel threads and lightweight threads multiplexed over time. Time is the synchroniser.

There is a scheduler for kernel threads and a scheduler for lightweight threads.

# time dimension scan scheduling

Have a shared buffer of events that all threads use.

Or a buffer of pointers for events that are dependent on other threads.

When there is a temporal dependency between events you mark the events with completion status.

Perhaps a bloom filter for a hierarchical nature of temporal dependency.

Each thread has a head and tail pointer into the events ring buffer.

If the completion of an event finishes, you set a flag to say that completion has finished. This means that all threads reset their tail pointers to 0 to scan from the beginning again to process events that weren't ready.




When an event meets it's dependencies, you are ready to schedule execution of that event.

You can sort the event buffer too.

The data should already be in memory.

For simplicity could use insertion sort Or quicksort or mergesort in parallel.

When does a sort occur?

Events are inserted as fast as they can be inserted.

But at some point events need to be sorted and scheduled and multiplexed.

The ring buffer is in one of two states, READING and WRITING.

When all threads have finished reading, the RingBuffer changes states to WRITING and all queued updates are made to the RingBuffer

After all events are enqueued, the RingBuffer is sorted and the RINGBUFFER changes to READING.

Only one thread needs to SORT. Push, pop shall block until the state changes to the right state.

# Sort stopping
