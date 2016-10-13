# Lecture 6: Swapping, Paging, and Virtual Memory

### Swapping
  + Not enough RAM?
    + keep some of their memory *elsewhere* (maybe on a disk)
      + but disk cannot run code, get register, etc.
  + Simple Swapping to Disk?
    + When a process yields, copy its memory to disk
    + When a process is scheduled, copy back to memory
    + With relocation hardware, we can put memory in different RAM locations
    + Each process could see a memory space as big as the total amount of RAM
    + Problem:
      + cost of context switch VERY high (slow)
      + still limiting processes to the amount of RAM we actually have

### Paging
  + Divide physical memory into units of a single fixed size
    + pretty small one, called __page frame__
  + Treat the virtual address space in the same way
  + store each virtual address space page's data in one physical address page frame
  + Use per-page translation to convert virtual to physical pages
  + Fragmentation?
    + Only internal fragmentation (NO external)
    + BUT since pages are small, the internal fragmentation is small
    + Just like cutting already created segment into smaller pieces
  + How to translate?
    + Using hardware (so that it would be fast)
      + Memory Management Unit (MMU)
        + MMU used to sit between the CPU and bus, but now they are typically integrated to CPU
      + Page table too big
        + So it would be stored in memory (not fast enough)
        + BUT we have a very fast set of MMU registers used as a cache
      + MMU and Multiple Process?
        + each process needs its own page table...
      + Ongoing MMU Operations
        + if the current process adds or removes pages
          + Directly update active page table in memory
          + Privileged instruction to flush cached entries
        + if we switch from one process to another
          + Maintain separate page tables for each process
          + Privileged instruction loads pointer to new page table
          + A reload instruction flushes previously cached entries
        + share pages between multiple processes
          + Make each page table point to same physical page
          + Can be read-only or read/write sharing

### Demand Paging
  + Why?
    + We want to support processes with large address space... but they are too big for the RAM
  + Definition?
    + move pages onto and off of disk "*on demand*"
      + only need to put those pages *actually referenced* to RAM (no need to load up all the pages and later get rid of all of them)
      + if a program needs a page, it will reference it, then (if not in memory) we just move it from disk to RAM
  + How?
    + MMU support "*not present*" pages
      + generated a fault/trap when they are referenced
      + OS brings in pages, update page table, and retry the reference (during which the program is blocked)
    + process starts only with a *subset* of its pages and add pages on demand
  + Performance?
    + if most memory references require disk access, TERRIBLE performance...
    + need to ensure that the page holding the next memory reference is already there
    + Solution: rely on __Locality of Reference__
      + the next address you ask for is likely to be close to the last address you asked for
      + Why locality of reference *usually present*?
        + code usually executes sequences of *consecutive or nearby* instructions
          + Most branches tend to be relatively short distances (into code in the same routine)
        + We typically need access to things in the current or previous stack frame
        + Many heap references to recently allocated structures
        + (no guarantee for data area, but few people writes program without locality of reference)
  + __Page Faults__
    + generated when pages *not in RAM* at this moment
    + handling:
      + initialize such entries to "not present" when constructing page table
      + CPU faults if "not present" page is referenced
        + enters kernel, forward handler
        + determine which page and where
        + schedule I/O to fetch it, block the process (other process can run)
        + read in page
        + back up user-mode PC to retry failed instruction
        + return to user-mode and try again
    + Page faults only slow a process down, __NEVER crashing the system__ (no impact on correctness)
    + Overhead:
      + process is *blocked* while reading pages, delaying execution and consuming cycles (proportional to the number of page faults)
      + Optimization: having "*right*" pages in memory (to have few faults)
        + cannot control what we read in, BUT can choose what we *kick out*
  + Page and Secondary Storage
    + when not in memory, pages live in __Swap Space__ on secondary storage (typically a disk)
    + How to manage?
      + As a pool of variable length partitions?
      + As a random collection of pages?
      + As a file system?
  + Virtual Memory Revisited
    + A generalization of what demand paging allows
    + A form of memory where the system provides a useful abstraction
      + A very large quantity of memory for each process
      + All directly accessible via normal addressing at a speed approaching that of actual RAM
    + Basic concepts
      + Give each process an address space of immense size
      + Allow processes to request segments within that space
      + Use dynamic paging & swapping to support the abstraction
    + Key issue:
      + How to create the abstraction when you don’t have that much real memory...
    + Key VM Technology:
      + Replacement algorithms: __Locality of Access__
        + find a page in RAM used in the distant past
        + replace it with the newly fetched page
  + Page Replacement
    + Basics:
      + keep some set of possible pages in memory
      + occasionally (e.g. page fault), we need to replace one of them with another page on disk
      + Paging hardware and MMU translation allows us to choose any page for ejection to disk
    + __Optimal Replacement Algorithm__
      + Replace the page that will be next referenced __furthest__ in the future
        + since it delays the next page fault as long as possible
      + Problem: how can we *predict*?
    + __Approximation__ to Optimal Replacement Algorithm
      + with __Locality of Reference__
        + note which pages have recently been used
        + use this data to predict future behavior
        + if locality of reference holds, the page will be accessed again soon
      + Candidates:
        + NO Random NO FIFO
        + Least Frequently Used: not actually useful
        + __Least Recently Used__
  + __Least Recently Used (LRU)__
    + *Naive* version:
      + When a page is accessed, record the time (need to store *timestamps* somewhere)
      + When you need to eject a page, check all timestamps for pages in memory (*heavy performance penalty*)
      + Choose the one with the oldest timestamp
      + Problem: where to keep timestamp?
        + MMU?
          + translation must be blindingly fast, getting/storing time on every fetch is VERY expensive
        + Software?
          + Mark all pages invalid, even if in memory
          + Take a fault first time each page is referenced, note the time
          + Then mark this page valid for the rest of the time slice
          + BUT this is meaningless... (Causing page faults to reduce the number of page faults...)
        + We need cheaper software surrogate for LRU...
    + __Clock Algorithm__
      + Organize all pages in *circular list*
      + MMU sets a __reference bit__ for the page on access
      + Scan whenever we need another page
        + For each page, ask MMU if page has been referenced
        + If so, reset the reference bit in the MMU & skip this page
        + If not, consider this page to be the least recently used
        + Next search starts from __exactly this position__, not head of list (another *guess pointer*)
      + No extra page faults, usually scan only a few pages
      + Compare True LRU and Clock Algorithm
        + Both are just approximations to the optimal
        + LRU clock’s decisions are 98% as good as true LRU
        + Clock algorithm MUCH faster
    + Page Replacement and Multiprogramming
      + We don’t want to clear out all the page frames on each context switch
      + Possible Choices
        + Single global pool?
          + Treat the entire set of page frames as a shared resource and approximate LRU for the entire set
          + Problem:
            + Bad interaction with *round-robin scheduling*: The one last in the queue will find all his pages swapped out, not because he isn’t using them. When he gets in, lots of page faults...
        + Fixed allocation of page frames per process?
          + wasting memory... not all process needs same amount of page frames
        + __Working set-based page frame allocations__
          + Working set: set of pages used by a process in a fixed length sampling window in the immediate past
          + Allocate enough page frames to hold each process’ working set; each process runs replacement within its own set
          + Optimal Working Set: number of pages needed during next time slice
            + too small? run slowly
            + too large? no great improvements
            + How to know?
              + by observing the process' behavior
          + Implementation:
            + Manage the working set size
              + Assign page frames to each in-memory process
              + Processes page against themselves in working set
              + Observe paging behavior (faults per unit time)
              + Adjust number of assigned page frames accordingly
            + __Page stealing algorithms__, e.g. __Working Set-Clock__
              + Track last use time for each page, for owning process
              + Find page least recently used (by its owner)
              + Processes that need more pages tend to get more
              + Processes that don't use their pages tend to lose them
          + Problem: __Thrashing__
            + Sum of working sets exceeds available memory, thus no one will have enough pages in memory
            + Whenever anything runs, it will grab a page from someone else
            + So they’ll get a page fault soon after they start running
            + Prevention:
              + DO NOT squeeze working set sizes
              + reduce number of competing processes by swapping some ready process out
        + Unswapping a Process
          + when a swapped process comes in from disk
            + Pure swapping?
              + Bring in all pages before process is run, no page faults
            + Pure demand paging?
              + Pages only brought in as needed; fewer pages per process, more processes in memory
            + Pre-loaded the last working set?
              + Far fewer pages to be read in than swapping
              + Probably the same disk reads as pure demand paging
              + Far fewer initial page faults than pure demand paging
        + __Clean VS Dirty Pages__
          + if the in-memory copy has not been modified, there is still a valid copy on disk
            + in-memory copy is "Clean"
            + can be replaced at any time
          + if the in-memory copy has been modified, the copy on disk is no longer up-to-date
            + in-memory copy is "Dirty", if swapped out of memory, *must be written to disk*
            + must be written to disk before the frame can be reused
          + Could only kick out clean pages
          + *Pre-emptive Page Laundering*
            + Ongoing background write-out of dirty, not running pages, making them clean again
        + Paging and Shared Segments
          + Some memory segments will be shared
          + Created/managed as mappable segments
          + Shared pages don't fit working set model
