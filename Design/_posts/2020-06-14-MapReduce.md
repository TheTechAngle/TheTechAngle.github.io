## MapReduce: Simplified Data Processing on Large Clusters

MapReduce is a programming model that allows users to specify a map function that processes a key-value pair to generate 
a set of intermediate key/value pairs and a reduce function that merges all intermediate values associated with the same key
[Link to paper](https://storage.googleapis.com/pub-tools-public-publication-data/pdf/16cb30b4b92fd4989b8619a61752a2387c6dd474.pdf)

"Our abstraction is inspired by the map and reduce primitives present in Lisp
and many other functional languages. We realized that
most of our computations involved applying a map operation to each logical “record” in our input in order to
compute a set of intermediate key/value pairs, and then
applying a reduce operation to all the values that shared
the same key, in order to combine the derived data appropriately"

### Execution Overview

1. The MapReduce library in the user program first
splits the input files into M pieces of typically 16
megabytes to 64 megabytes (MB) per piece. It
then starts up many copies of the program on a cluster of machines.

2. One of the copies of the program is special – the
master. The rest are workers that are assigned work
by the master. There are M map tasks and R reduce
tasks to assign. The master picks idle workers and
assigns each one a map task or a reduce task.

3. A worker who is assigned a map task reads the
contents of the corresponding input split and passes each
pair to the user-defined Map function. 
The intermediate key/value pairs produced by the Map function
are buffered in memory.

4. Periodically, the buffered pairs are written to local
disk, partitioned into R regions by the partitioning
function. The locations of these buffered pairs on
the local disk are passed back to the master, who
is responsible for forwarding these locations to the
reduce workers.

5. When a reduce worker is notified by the master
about these locations, it uses remote procedure calls
to read the buffered data from the local disks of the
map workers. When a reduce worker has read all intermediate data, it sorts it by the intermediate keys
so that all occurrences of the same key are grouped
together. The sorting is needed because typically
many different keys map to the same reduce task. If
the amount of intermediate data is too large to fit in
memory, an external sort is used.

6. The reduce worker iterates over the sorted intermediate data and for each unique intermediate key encountered, 
it passes the key and the corresponding set of intermediate values to the user’s Reduce function. 
The output of the Reduce function is appended to a final output file for this reduce partition.
When a reduce task completes, the reduce worker atomically renames its temporary output file to the final output file

### Fault Tolerance

The master pings every worker periodically. Any map tasks completed by the worker are reset back to their initial idle state,
and therefore become eligible for scheduling on other workers.

### Master Failure

It is easy to make the master write periodic checkpoints of the master data structures described above. If the master task dies, a new copy can be started from the last
checkpointed state. However, given that there is only a single master, its failure is unlikely; therefore our current implementation aborts the MapReduce computation
if the master fails.

### Locality

We conserve network bandwidth by taking advantage of the fact that the input data is stored on the local disks of the machines that make up our cluster.
The MapReduce master takes the location information of theinput files into account and attempts to schedule a map
task on a machine that contains a replica of the corresponding input data. Failing that, it attempts to schedule
a map task near a replica of that task’s input data.

### Task Granularity

We subdivide the map phase into M pieces and the reduce phase into R pieces, as described above. Ideally, M
and R should be much larger than the number of worker machines. Having each worker perform many different
tasks improves dynamic load balancing, and also speeds up recovery when a worker fails: the many map tasks
it has completed can be spread out across all the other worker machines.

### Backup Tasks

One of the common causes that lengthens the total time taken for a MapReduce operation is a “straggler”: a machine 
that takes an unusually long time to complete one of the last few map or reduce tasks in the computation.

When a MapReduce operation is close to completion, the master schedules backup executions
of the remaining in-progress tasks. The task is marked as completed whenever either the primary or the backup execution completes.

### Refinements

1) Partitioning function

Data gets partitioned across these tasks using a partitioning function on the intermediate key. A default partitioning function is
provided that uses hashing (e.g. “hash(key) mod R”).

2) Ordering Guarantees
We guarantee that within a given partition, the intermediate key/value pairs are processed in increasing key order.

3) Combiner Function

Combining some map results it needed before sending it over to reduce, like an intermediate reduce. For word count for example you
could have millions of <the - 1> pairs

4) Local Execution

To help facilitate debugging, profiling, and small-scale testing, we have developed an alternative implementation of the MapReduce library that sequentially
executes all of the work for a MapReduce operation on the local machine.

