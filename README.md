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
