# Lecture 4: Scheduling

### What is?
  * A resource that can serve one client at a time
  * We have multiple clients
  * Who use next? For how long?

### How To?
  * Decision depend on __Goals__
    + with related algorithm
    + optimizing different quantities
      * drastically change system behavior

  * __Goals?__:
    + Maximize throughput (more work done in a certain period)
    + Minimize average waiting time
    + Fairness
    + Priority
    + Real time (meaning with *deadline* connected to real world time)
      * Extra:
        + Time sharing = fast response time + equal CPU share
        + Batch = maximize throughput __-__ individual delay worriness (ignoring)
        + Real time = Critical happen in time + non-critical may not happen

### Body?
  * __Process Queue__ (Processes __Ready__ to run)
    + Ordered by which will run next based on algorithm (first one of the queue will be the next to run) (better NOT policy-dependent)
      * What is *Not Ready*?
        + not in queue or at the end of queue
        + ignored by the scheduler

### Types?
  + __Preemptive__
    + Interrupt running process, let others use for a while
      + Pros:
        + Good response time
        + VERY fair!
        + YES real time YES priority scheduling
      + Cons:
        - LOW throughput
        - NO simple
        - Halt process cleaning & save process state, DIFFICULT!
  * __Non-Preemptive__
    + Scheduled work use resource till the end (no interruption)
      + Pros:
        + Low overhead
        + High throughput
        + simple
      + Cons:
        - POOR response time
        - Bugs!!! (e.g. infinite loop, surprise!)
        - NO fair!
        - X real time X priority scheduling

### Policy & Mechanism?
  * __Dispatching__: *move* jobs into and out of a processor
    + Better separate the choice of who runs (policy) from the dispatching mechanism (How it is done should NOT depend on policy)

### Performance?
  * Schedule impact Performance (greatly)
    + You CANNOT optimize all of performance aspects, sadly
    + performance has very different characteristic under light vs. heavy load

  + Performance Goals
    + What kind of goal?
      + Quantitative (more? less? equal? how much?)
      + Measurable
        + need to define a process for measuring
    * So We choose WHAT?
      + Candidates:
        + Throughput? (w/ processes/second)
          + Different processes need different run times
          + Completion time? Not scheduler's fault...
        + Delay? (milliseconds)
          + Which Delay?
            + Time to finish a job (Turnaround time)?
            + Response time?
            + Others (not scheduler's fault): e.g. Time to complete a service request, time to wait for a busy resource
        + Others?
          + Mean time to completion (seconds)
          + Throughput (operations per second)
          + Mean response time (milliseconds)
          + Overall “goodness” (customer-specific, usually stated in Service Level Agreements)

### EXAMPLES? CHECK SLIDES!!!!!!

### Graceful Degradation?
  + __Overload__: when under too much load, system can no longer meet service goals
    + Now what?
      + Continue with degraded performance
      + Reject work to maintain performance
      + When load back to normal, back to normal service
      + DO NOT allow throughput \-\-> 0!
      + DO NOT allow response time \-\-> ∞!

### Non-Preemptive (Again)?
  + Scheduled process runs until it __yields__ (finish using, free) CPU
  + Good for simple systems w/ *few processes & natural producer-consumer relationships*
  + Good for maximizing throughput (why mentioned again?)
  + Process has to yield by *themselves/voluntarily*
  + (else, piggy process starve others & buggy process lock up system)

  + __Algorithms:__
    * __First Come First Served (FCFS)__
      + Run first process on queue until it completes or yields
      + Run next process on queue until...
      + Simplest
      + Delays highly variable
      + All process will eventually be served
      * Works well w/:
        + batch systems (response time NOT important)
        + embedded (e.g. telephone) systems (computations brief and/or exist in natural producer/consumer relationships)
    * __Shortest Job Next (SJN)__
    * __Real Time Schedulers__
      + some things must happen at particular times (with real-time deadlines)
      + Hard Real Time Scheduler:
        + The system must meet its deadlines, otherwise system fails
        + To avoid missing deadline, we must have:
          + pre-defined schedule (typically)
          + scheduler follows strictly
          + deep understanding of the code in jobs
          + avoid non-deterministic timings
          + turn OFF interrupts (typically)
          + Non-Preemptive (typically)
      + Soft Real Time scheduler:
        + *highly desirable* to meet your deadlines, but *allow (some/any) missing*
        + goal: avoid missing deadlines
        + may have different classes of deadlines
        + need not require quite as much analysis
        + about Non-Preemptive:
          * not as important tasks meet their deadline
          * not as predictable, LESS careful analysis
          * if a new task with earlier deadline arrives, non-preemptive would possibly fail the deadline
        + __Algorithms: Earliest Deadline First__
          + job has a deadline
          + queue sorted by deadline
          + pick first in queue
          + remove jobs with missed deadlines in queue
          + minimizes total lateness
          + e.g. video playing device (missing frames? Fine. Skip them)
      + Missing Deadline?
        + result depends on different systems (well defined)
        + some may just drop, or allow falling behind, or drop future jobs

### Preemptive (Again)?
  + __Clearer Definition__
    + A thread or process is chosen to run till it
      + *yields*
      + OR OS decides to *interrupt* it w/ other process/thread runs
    + interrupted process/thread is *restarted later* (typically)
  + Implication
    + a process can be forced to yield at any time
      + if a higher priority process is ready (e.g. IO completion interrupt)
      + if running process’s priority is lowered (e.g. running too long)
    + interrupted process might not be in a "clean" state (saving & restoring state NOT EASY)
    + enables "fair share" scheduling
    + have *context switches*
    + potential resource sharing problems
  + Implementation
    + Need to...
      + get control away from process, using e.g.
        + system calls
        + __clock interrupt__
          + processors have clocks (peripheral)
          + generate an interrupt at fixed interval that halts any running process
          + ensure that runaway process doesn’t keep control forever & __key technology__ for preemptive scheduling
      + consult scheduler before returning to process
        + any ready process had priority raised?
        + any process been awakened?
        + current process had its priority lowered?
      + scheduler finds highest priority ready process
        + if == current, return;
        + else, yield on behalf of current process and switch to higher priority process
    + __Algorithm: Round Robin Scheduling Algorithm__
      + Goal: fair share scheduling
        + all processes have *equal shares* of CPU
        + all processes have *similar queue delays*
      + Steps:
        + All processes are assigned a nominal __time slice__
          + Usually the same sized slice for all
          + __Time Slice__
            + performance depends heavily on how long time slice is
              + long time slices avoid too many context switches that wastes cycles, having better throughput
              + short time slices have better response time
              + need balance
            + cost:
              + What % of CPU use does a process get?
              + More processes in queue = fewer slices/second
              + CPU share = time_slice * slices/second
              + Natural rescheduling interval
                + When a process blocks for resources or I/O
                + Ideally, fair-share would be based on this period
                + Time-slice ends only if process runs too long
        + Each process is scheduled in turn
          + run till it blocks, or its time slice expires, then put at the end of process queue
        + Then the next process is run
          + eventually, each process reaches front
      + Properties:
        + processes get quick chance to do SOME computation
          + cost: not finishing process as quickly
          + good for interactive processes
        + FAR MORE context switches
        + Runaway (infinite) processes do LITTLE harm
      + Interrupts:
        + processes halted if time slice expires
        + if block for I/O (or anything else) on their own, doesn’t halt them, thus sometimes round robin acts no differently than FIFO
      + __Example: VIEW SLIDES__
      + Context Switch Cost:
        + Entering the OS (interrupt, save registers, call scheduler)
        + Cycles to choose who to run (by scheduler/dispatcher)
        + Moving OS context to the new process (switch stack, non-resident process description)
        + Switching process address spaces
        + Losing instruction and data caches (greatly slowing down the next hundred instructions)

### Multi-Level Feedback Queue (MLFQ) Scheduling
  + Why?
    + One time slice length may not fit all processes
  + Details:
    + Create multiple ready queues
      + Short quantum (foreground) tasks that finish quickly w/ short and frequent time slice, optimize *response time*
      + Long quantum (background) tasks that run longer w/ longer but infrequent time slices, *minimize overhead (good turnaround time)*
    + Different queues may get different shares of the CPU
  + How to put to queue?
    + Start all processes in short quantum queue
      + Move *downwards* if too many time-slice ends
      + Move back *upwards* if too few time slice ends
      + Processes dynamically find the right queue
    + For real time task, start in real time queue and don’t move

### Priority Scheduling Algorithm
  + Sometimes processes aren’t all equally important, and we might want to run the more important ones first
  + How?
    + Assign each job a __priority number__
    + Run according to priority number
  + Priority and Preemption
    + Non-preemptive:
      + priority scheduling is just about ordering processes w/ highest priority first (similar to Short Job First)
    + Preemptive:
      + when new process w/ higher priority is created, it might preempt running process
  + Problems
    + Possible starvation
    + Can a low priority process *ever* run?
    + If not, is that the effect we wanted?
  + Solution to problem:
    + Adjust priorities
      + Processes that have run for a long time have priority temporarily *lowered*
      + Processes that have not been able to run have priority temporarily *raised*
  + Hard Priority & Soft Priority:
    + Hard: higher priority has absolute precedence over the lower
    + Soft: higher priority gets a larger share of resource than the lower
  + Use in Linux:
    + "nice value" (Soft)
    + Commands can be run to change process priorities
    + Anyone can request lower priority for his processes
    + Only privileged user can request higher
  + Use in Windows:
    + 32 different priority levels
    + Users can choose from 5 of them
    + Kernel adjusts priorities based on process behavior
