COPY ON WRITE FORK in xv6

Modified Files:

1. vm.c
- uvmcopy: used mappages to map the PTE with the same pa
- cow_fault: copies the content of the shared page to newly allocated page
- copyout: This checks for cow_fault and then copies out
  
2. kalloc.c
- added a refcount arary
- inc_ref & dec_ref: function that increase and decrease the ref count array for particular
index based on physical address pa
- kfree: every time kfree is called it will call dec_ref and if refcount becomes zero then it
frees the page
- kalloc: intialization of refcount of particular index based on physical address pa to 1
  
3. trap.c
- checks for pagefault if it happens then it calls the cow_fault to check pagefault happend
because of cow if it is then it kills the process

ANALYSIS:
To reord the frequency of page faults i have taken a variable no_of_pagefaults in proc struct
and intilazed them to 0 then every time a page fault happens in the user trap i have incremented the
variable

I have used the forks given in lazytest

READ ONLY : Simple Fork

Number of Page Faults = 0

WRITE ONLY: Three Fork

Number of Page Faults =6554

In a Copy- On- Write (COW) fork, page faults occur when a process modifies shared memory,
triggering the creation of a new memory page. The results show that with read- only processes,
there are no page faults, as pages are shared without modification. Whereas, write- only processes
result in 6554 page faults, as each modification requires a new page allocation. This demonstrates
COW's efficiency for read- only operations and the expected increase in page faults for write
operations to maintain memory isolation.

Number of Times COW Mechanism Called:

READ ONLY : Simple Fork

Number of times cow mechanism called is 17

WRITE ONLY: Three Fork

Number of times cow mechanism called =16197 

For read only process cow mechanism is called 17 times (all in copyout) and for write only process
cow mechanism is 16197 (6554 times in usertrap 9643 times in coyout)

Memory Conservation:

Reduced Initial Copies: In a typical fork, each of the n processes with m pages requires all n×m
pages to be copied. With Copy- On- Write (COW), these pages remain shared initially, so no copies
are made until a modification is necessary.

Selective Copying: 

COW only duplicates pages when a process modifies them. For many processes
that do little or no writing (such as those that immediately call exec), only a small portion of the
n×m pages need to be copied.

Efficient for Lightweight Forks:

COW is particularly efficient for processes that don’t modify
much memory after forking, as only modified pages are duplicated, conserving memory.
COW allows the parent and child to share memory pages rather than creating separate copies
immediately. This reduces the initial memory overhead because no additional memory is consumed
for the duplicated pages as long as they remain unchanged.

Efficiency:

Reduced Memory Usage: COW fork allows the parent and child processes to share memory pages
until a modification is made, conserving memory by avoiding unnecessary duplication.
Faster Forking Process: Since memory pages aren’t copied immediately, the fork operation
completes faster, reducing the overhead associated with starting new processes.
On- Demand Duplication: COW only copies pages when a process writes to them, so for processes
that perform minimal or no modifications, very few pages are actually duplicated.
Ideal for Short- Lived Processes: Processes that quickly call exec or perform limited writes
benefit greatly from COW, as little to no extra memory is allocated, enhancing system efficiency.
Improved Scalability: With COW, the system can handle more concurrent processes with lower
memory usage, supporting better scalability in multi- process environments.

FURTHER OPTIMIZATIONS:

Further optimizations for the Copy- On- Write (COW) fork can boost memory efficiency and system
performance
Delayed Page Allocation: Instead of copying pages on the first write, COW could delay allocations
by batching writes or grouping them. This approach would save both memory and time by reducing
the number of page copies, especially when multiple writes happen close together.
Efficient Page Tracking: By using finer- grained tracking for modified sections of pages, the
system could avoid copying entire pages when only small parts are modified. This would be
particularly useful for processes that make localized changes, improving memory efficiency.
Page Sharing Across Forks: For processes that fork and read the same static data or shared
libraries, the system could maintain shared pages across multiple forks. This would further conserve
memory, especially for data that changes rarely, such as read- only resources like libraries or
configuration files.
