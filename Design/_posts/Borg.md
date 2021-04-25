# Borg

## User perspective

Clusters and Cells
* The machines in a cell belong to a single cluster.
* Jobs can have constraints to force its tasks to run on machines with particular attributes such as processor architecture, OS version
* Borg programs are statically linked to reduce dependencies on their runtime environment, and structured as packages of binaries and data files, whose installation is orchestrated by Borg.
* Every job has a priority, a small positive integer. A high priority task can obtain resources at the expense of a lower priority one 
* We disallow tasks in the production priority band to preempt one another.
* Borg creates a stable “Borg name service” (BNS) name for each task that includes the cell name, job name, and task number stored in  a consistent,
highly-available file in Chubby 
* The BNS name also forms the basis of the task’s DNS name, so the fiftieth task in job jfoo owned by user ubar in cell cc would be reachable via 50.jfoo.ubar.cc.borg.google.com
* Almost every task run under Borg contains a built-in HTTP server that publishes information about the health of the task and thousands of performance metrics
* A service called Sigma provides a web-based user interface (UI) through which a user can examine the state of all
their jobs

## Borg Architecture

Borgmaster
* The main Borgmaster process handles client RPCs that either mutate state (e.g., create job) or provide read-only access to data (e.g., lookup job). It also manages state machines for all of the objects in the system (machines, tasks, allocs, etc.), communicates with the Borglets, and offers a web UI as a backup to Sigma.
* replicated 5 times, each replica maintains an inmemory copy of most of the state of the cell, and this state is also recorded in a highly-available, distributed, Paxos-based store 
* A master is elected (using Paxos) when the cell is brought up and whenever the elected master fails; it acquires a Chubby lock so other systems can find it.
* A high-fidelity Borgmaster simulator called Fauxmaster contains a complete copy of the production Borgmaster code and we use it to debug failures

Scheduling 
* the Borgmaster records a job persistently in the Paxos store and adds the job’s tasks to the pending queue
* The scan proceeds from high to low priority, modulated by a round-robin scheme within a priority to ensure fairness across users and avoid head-of-line blocking behind a large job
* In feasibility checking, the scheduler finds a set of machines that meet the task’s constraints and also have enough “available” resources – which includes resources assigned to lower-priority tasks that can be evicted.
* Borg originally used a variant of E-PVM [4] for scoring,  E-PVM ends up spreading load across all the machines, leaving headroom for load spikes – but at the expense of increased fragmentation (worst-fit)
*  “best fit”, which tries to fill machines as tightly as possible. This leaves some machines empty of user jobs  but the tight packing penalizes any mis-estimations in resource requirements
* Current scoring model is a hybrid one that tries to reduce the amount of stranded resources – ones that cannot be used because another resource on the machine is fully allocated.
* We add the preempted tasks to the scheduler’s pending queue, rather than migrate or hibernate them
* To reduce task startup time, the scheduler prefers to assign tasks to machines that already have the necessary packages
installed: most packages are immutable and so can be shared and cached. (This is the only form of data locality supported by the Borg scheduler.)
* In addition, Borg distributes packages to machines in parallel using treeand torrent-like protocols.

Borglet
* The Borglet is a local Borg agent that is present on every machine in a cell. It starts and stops tasks; restarts them if they fail; manages local resources by manipulating OS kernel settings; rolls over debug logs; and reports the state of the machine to the Borgmaster and other monitoring systems
* For performance scalability, each Borgmaster replica runs a stateless link shard to handle the communication with some of the Borglets
* the Borglet always reports its full state, but the link shards aggregate and compress this information by reporting only differences to the state machines, to reduce the update load at the elected master


Scalability
* A single Borgmaster can manage many thousands of machines in a cell, and several cells have arrival rates above 10 000 tasks per minute. A busy Borgmaster uses 10–14 CPU cores and up to 50 GiB RAM. We use several techniques to achieve this scale.
* To handle larger cells, we split the scheduler into a separate process so it could operate in parallel with the other Borgmaster functions
* A scheduler replica operates on a cached copy of the cell state. It repeatedly: retrieves state changes from the elected master (including both assigned and pending work); updates its local copy; does a scheduling pass to assign tasks; and informs the elected master of those assignments
* Several things that make the Borg scheduler more flexible
  * Score Caching:- Caching the score given to a machine for its feasibility. Cached until something on the machine or task changes i.e., a task terminates, a task requirements change
  * Equivalence:- Borg only does feasibility and scoring for one task per equivalence class – a group of tasks with identical requirements.
  * Relaxed randomization:-  It is wasteful to calculate feasibility and scores for all the machines in a large cell, so the scheduler examines machines in a random order until it has found “enough” feasible machines to score, and then selects the best within that set. **Scheduling a cell’s entire workload from scratch typically took a few hundred seconds, but did not finish after more than 3 days when this techniques were disabled**
