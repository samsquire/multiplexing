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

Have a shared thread safe ringbuffer of events that all threads use.

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

# per thread state

Each kernel or green thread has a buffer of pending written events and pending requests that they are waiting to read and pending event dequeuing.



On each re-entrance tick each thread/green thread greedily dequeues as many work items as it can and adds it to its read buffer. It then tries to enqueue its write buffer into the event RingBuffer.

It then processes events in its read buffer.

The thread uses the non blocking calls to the event ringbuffer.

We want threads to all be READING at the same time and WRITING at the same time, so we need some way of marking a thread's status as finished doing either of these.

At the beginning of a thread/green thread tick, we call eventbuffef.block() and one of the thread checks that all threads are FINISHED_WRITING and if so, it compareAndSet the event ringbuffer status to SORTING. The first successful thread then sorts the event buffer. At the end of the sort the thread status changes to READING state of the event ringbuffer.

When a thread has enqueued all its reads to the event ringbuffer it marks itself as FINISHED_READING. It then calls block() on the event ringbuffer which waits until all threads have FINISHED READING.

It then marks itself as WRITING and greedily Enqueues all its writes. Then it marks itself as FINISHED_WRITING and proceeds to process events.

When a thread is in processing mode, it can queue up modifications to data structures for action in WRITING_DATA mode.

WRITING DATA serialises modifications to in memory data structures so all threads get the same perspective of in memory data.

This works by a compareAndSwap on the event ringbuffer for mode WRITING_DATA.

Only one thread shall succeed on setting this flag. Those threads that fail shall block retrying.

How to enforce ordering of WRITING DATA changes?

Ideally you want the WTITING DATA to be in order of requests.

We could produce a synthetic event for writing data that would be sorted.

This shall be sorted.

We can set a local flag of BLOCKED on the green thread when we dequeue an event that is of kind WRITING DATA that is for a thread that is not us.

Then if a thread dequeeues a WRITING DATA event during READING phase, it processes the writes.

After the READ phase we eventbuffer.block() on one thread finished WRITING or all FINISHED READING.

This serialises and orders the WRITING DATA.

There is therefore the following phases:
 * Blocking=False
 * EVENT BUFFER READING
 * If a WRITING DATA is for a thread that is not us or were not interested in, set Blocking = true
 * One thread detects it's head of the queue for writing data and does data modification in memory. It's guaranteed to be the only thread modifying data in order. Sets thread status to FINISHED WRITING DATA
 * EVENT BUFFER.BLOCK(waitingFor=all threads FinishedReading||FinishedWritingData
 * EVENT BUFFER WRITING
 * Remember the head position. We shall use this to implement incremental sort.
 * All threads enqueue to event ringbuffer
 * EVENT BUFFER.BLOCK(waitingFor=FinishedWriting)
 * The last thread decided to sort the RingBuffer. The head and tail might not need to change if we only sort new inserted events coming in.
 * All threads process events, queueing up WRITE events, WRITING DATAs events

We need to mark events as processed when finished. We skip enqueuing events to local read buffers that are finished. This gives us error handling.

# load balancing and work stealing

By merit of everyone dequeuing at the same time and everyone writing at the same time, we can implement work stealing in the scheduler.

As a process is dequeuing items in the READ stage it can create a local buffer of the queues of each thread.

If the local thread has no work, it can change the owner of half of the work in that thread's queue.

Load balancing.

We can use the modulo operator to decide if some work is for a thread.

How to stop threads fighting over work stealing? We can introduce a phase that only one thread can enter called WORK_STEAL. This is a compare and swap on the event ringbuffer.

Every other thread that fails to set it, doesn't matter.

All threads shall block waiting until the work stealing is complete.

# Sort stopping
