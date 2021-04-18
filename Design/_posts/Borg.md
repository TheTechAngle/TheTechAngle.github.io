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
