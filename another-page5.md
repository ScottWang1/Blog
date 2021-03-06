---
layout: default
---

## The Garbage Collection 

## Mark-sweep
The first algorithm that we look at is mark-sweep collection. It is a straightforward embodiment of the recursive definition of pointer reachability. Collection operates in two phases. First, the collector traverses the graph of objects, starting from the roots (registers, thread stacks, global variables) through which the program might immediately access objects and then following pointers and marking each object that it finds. Such a traversal is called tracing. In the second, sweeping phase, the collector examines every object in the heap: any unmarked object is deemed to be garbage and its space reclaimed.
Mark-sweep is an indirect collection algorithm. It does not detect garbage per se, but rather identifies all the live objects and then concludes that anything else must be garbage. Note that it needs to recalculate its estimate of the set of live objects at each invocation. Not all garbage collection algorithms behave like this. 
![图片pic1]({{ "/assets/images/2.1.png" | absolute_url }})
![图片pic1]({{ "/assets/images/2.2.png" | absolute_url }})

Termination of the marking phase is enforced by not adding already marked objects to the work list, so that eventually the list will become empty. At this point, every object reachable from the roots will have been visited and its mark-bit will have been set. Any unmarked object is therefore garbage.
![图片pic1]({{ "/assets/images/2.3.png" | absolute_url }})
The sweep phase returns unmarked nodes to the allocator (Algorithm 2.3). Typically,
the collector sweeps the heap linearly, starting from the bottom, freeing unmarked nodes
and resetting the mark-bits of marked nodes in preparation for the next collection cycle.
##### Bitmap marking
It can be improved by Bitmap marking. With a bitmap, marking will not modify any object, but will only read pointer fields of live objects. Other than loading the type descriptor field, no other part of pointer-free objects will be accessed. Sweeping will not read or write to any live object although it may overwrite fields of garbage objects as part of freeing them (for example to link them into a free-list). Thus bitmap marking is likely to modify fewer words, and to dirty fewer cache lines so less data needs to be written back to memory. Bitmap marking was also motivated by the concern to minimise the amount of paging caused by the collector
##### Lazy sweeping
We observe two properties of objects and their mark-bits. First, once an object is garbage, it remains garbage: it can neither be seen nor be resurrected by a mutator. Second, mutators cannot access mark-bits. Thus, the sweeper can be executed in parallel with mutator threads, modifying mark-bits and even overwriting fields of garbage objects to link them into allocator structures. The sweeper (or sweepers) could be executed as separate threads, running concurrently with the mutator threads, but a simple solution is to use lazy sweeping. Lazy sweeping amortises the cost of sweeping by having the allocator perform the sweep. Rather than a separate sweep phase, the responsibility for finding free space is devolved to allocate. At its simplest, allocate advances the sweep pointer until it finds sufficient space in a sequence of unmarked objects. However, it is more practical to sweep a block of several objects at a time.

Lazy sweeping offers a number of benefits. It has good locality: object slots tend to be used soon after they are swept. It reduces the algorithmic complexity of mark-sweep to be proportional to the size of the live data in the heap, the same as semispace copying collection
## Mark-compact garbage collection
The strategy we consider is in-place compaction of objects into one end of the same region. 
##### Two-finger compaction
This Agorithm starts with two pointers or 'fingers', free which points to the start of the region and scan which starts at the end of the region. The first pass repeatedly advances the free pointer until it finds a gap (an unmarked object) in the heap, and retreats the scan pointer until it finds a live object. If the free and scan fingers pass each other, the phase is complete. Otherwise, the object at scan is moved into the gap at free, overwriting a field of the old copy (at scan) with a forwarding address, and the process continue.
![图片pic1]({{ "/assets/images/3.1.png" | absolute_url }})
The benefits of this algorithm are that it is simple and fast, doing minimal work at each iteration, but the order of objects in the heap that results from this style of compaction is arbitrary, and this tends to harm the mutator's locality.
##### The Lisp 2 algorithm
The first pass over the heap (after marking) computes the location to which each live object will be moved, and stores this address in the object's forwardingAddress field (Algorithm 3.2). The computeLocations routine takes three arguments: the addresses of the start and the end of the region of the heap to be compacted, and the start of the region into which the compacted objects are to be moved. Typically the destination region will be the same as the region being compacted, but parallel compactor threads may use their own distinct source and destination regions. The computeLocations procedure moves two 'fingers' through the heap: scan iterates through each object (live or dead) in the source region, and free points to the next free location in the destination region. If the object discovered by scan is live, it will (eventually) be moved to the location pointed to by free so free is written into its f orwardingAddress field, and is then incremented by the size of the object . If the object is dead, it is ignored.

The second pass updates the roots of mutator threads and references in marked objects so that they refer to the new locations of their targets, using the forwarding address stored in each about-to-be-relocated object's header by the first pass. Finally, in the third pass, relocate moves each live (marked) object in a region to its new destination.
![图片pic1]({{ "/assets/images/3.2.png" | absolute_url }})
It makes three passes over the heap, each iteration does little work.The chief drawback of the Lisp 2 algorithm is that it requires an additional full-slot field in every object header to store the address to which the object is to be moved.
##### One-pass algorithms
Marking sets the bits corresponding to the first and last granules of each live object. For example, bits 16 and 19 are set for the object marked old in Figure 3.3. By scrutinising the mark bitmap in the compaction phase, the collector can calculate the size of any live object.
![图片pic1]({{ "/assets/images/3.3.png" | absolute_url }})
##### Summary
Mark-sweep garbage collection uses less memory than other techniques such as copying collection (which we discuss in the next chapter). Furthermore, since it does not move objects, a mark-sweep collector need only identify (a superset of) the roots of the collection; it does not need to modify them. Both of these considerations may be important in environments where memory is tight or where the run-time system cannot provide type-accurate identification of references.

As a non-moving collector, mark-sweep is vulnerable to fragmentation. Using a parsimonious allocation strategy like sequential-fits reduces the likelihood of of fragmentation becoming a problem, provided that the application does not allocate very many large objects and that the ratio of object sizes does not change much. However, fragmentation is certainly likely to be a problem if a general-purpose, non-moving allocator is used to allocate a wide variety of objects in a long-running application. For this reason, most production Java virtual machines use moving collectors that can compact the heap.
## Copying garbage collection
 Copying compacts the heap, thus allowing fast allocation, yet requires only a single pass over the live objects in the heap. Its chief disadvantage is that it reduces the size of the available heap by half.
![图片pic1]({{ "/assets/images/copy.png" | absolute_url }})
The main purpose of Copying garbage collection is to solve the debris generated by the mark-and-sweep algorithm.
##### Summary
Compare mark-sweep and copying collection. Mark-sweep collectors have twice as much usable heap space available as do copying collectors, and hence will perform half as many collections, all other things being equal. Consequently, we might expect that mark-sweep collection offer better overall performance. Blackburn found that this was indeed the case for collection in tight heaps, where he used a segregated-fits allocator for the non-moving collector. Conversely, in large heaps the locality benefits to the mutator of sequential allocation outweighed the space efficiency of mark-sweep collection, leading to better miss rates at all levels of the cache hierarchy. This was particularly true for newly allocated objects which tend to experience higher mutation rates than older objects.

Copying collection offers two immediately attractive advantages over non-moving
collectors like mark-sweep: fast allocation and the elimination of fragmentation (other than to
satisfy alignment requirements). Simple copying collectors are also easier to implement
than mark-sweep or mark-compact collectors. The trade-off is that copying collection uses
twice as much virtual memory as other collectors in order to match their frequency of collections.  
The immediate disadvantage of semispace copying is the need to maintain a second semi- space. For a given memory budget and ignoring the data structures needed by the collector itself, semispace copying provides only half the heap space of that offered by other whole heap collectors. The consequence is that copying collectors will perform more garbage collection cycles than other collectors. Whether or not this translates into better or worse performance depends on trade-offs between the mutator and the collector, the characteristics of the application program and the volume of heap space available.
![图片pic1]({{ "/assets/images/4.5.png" | absolute_url }})


Figure 4.5 show that copying collection can be made more efficient that mark-sweep collection, provided that the heap is large enough and r is small enough. 
## Reference counting
 Rather than tracing reachable objects and then inferring that all unvisited objects must be garbage, reference counting operates directly on objects as references are created or destroyed.
It maintains a simple invariant: an object is presumed to be live if and only if the number of references to that object is greater than zero.
![图片pic1]({{ "/assets/images/5.1.png" | absolute_url }})
Algorithm 5.1, reference counts are incremented or decremented as references to objects are created or destroyed.
##### Advantages and disadvantages of reference counting
Potentially, reference counting can recycle memory as soon as an object becomes garbage. Consequently, it may continue to operate satisfactorily in a nearly full heap, unlike tracing collectors which need some headroom.  
Since reference counting operates directly on the sources and targets of pointers, the locality of a reference counting algorithm may be no worse than that of its client program and it can be implemented without assistance from or knowledge of the run-time system. In particular, it is not necessary to know the roots of the program.It can reclaim some memory even if parts of the system are unavailable: this is particularly useful in distributed system  
Unique pointers ensure that an object has a single 'owner'. When this owner is destroyed, the object also can be destroyed. 

Unfortunately, there are also a number of disadvantages to reference counting. First, reference counting imposes a time overhead on the mutator.  
Second, both the reference count manipulations and the pointer load or store must be a single atomic action in order to prevent races between mutator threads which would risk
premature reclamation of objects.   
Third, naive reference counting turns read-only operations into ones requiring stores to memory. These writes 'pollute' the cache and induce extra memory traffic. 
##### Improving efficiency
Deferral: Deferred reference counting trades fine grained incrementality (the immediate recovery of all garbage) for efficiency. The identification of some garbage objects is deferred to a reclamation phase at the end of some period. These schemes eliminate some barrier operations.  
Coalescing: Many reference count adjustments are temporary and hence 'unnecessary'; programmers often remove them by hand. In some special cases, this can also be done by the compiler. However, it is also possible to do this more generally at run time by tracking only the state of an object at the beginning and end of some period. This coalesced reference counting ignores all but the first modification to a field of an object in each period.  
Buffering: Buffered reference counting also defers identification of garbage. However, unlike deferred or coalesced reference counting, it buffers all reference count increments and decrements for later processing. Only the collector thread is allowed to apply reference count modifications. Buffering considers when to apply reference count modifications not whether to.
##### Summary
Reference counting taxes every pointer read and write operation and thus imposes a much larger tax on throughput than tracing does. Furthermore, multithreaded applications require the manipulation of reference counts and updating of pointers to be expensively synchronised. This tight coupling of mutator actions and memory manager risks some fragility, especially if 'unnecessary' reference count updates are optimised away by hand. Finally, reference counts increase the sizes of objects.
Because it cannot reclaim an object until the last pointer to that object has been removed, it cannot reclaim cycles of garbage.  
Its drawbacks make simple reference counting uncompetitive as a general purpose memory management component of a virtual machine, especially if the majority of objects managed are small, cycles are common and the rate of pointer mutation is high.  
Reference counting can play well in a mixed ecology where the lifetimes of most objects are sufficiently simple to be managed explicitly. Furthermore, it can be implemented as part of a library rather than being baked into the language's run-time system. It can therefore give the programmer complete control over its use, allowing her to make the decision between performance overhead and guarantees of safety.
## Comparing garbage collectors
###### Pause time
In a study of twenty Java benchmarks and six different collectors, Fitzgerald and Tarditi found that for each collector there was at least one benchmark that would have been at least 15% faster with a more appropriate collector.  
The number of instructions executed to visit an object for mark-sweep tracing are fewer than those for copying tracing. Locality plays a significant part here as well.  
 Prefetching techniques could be used to hide cache misses. However, it is an open question as to whether such techniques can be applied to copying collection without losing the benefits to the mutator of depth- first copying.  
 If marking is combined with lazy sweeping, we obtain greatest benefit in the same circumstances that copying performs best: when the proportion of live data in the heap is small.
The tracing collectors considered so far have all been stop-the-world: all mutator threads must be brought to a halt before the collector runs to completion, and we saw that deferred and coalesced reference counting, the most effective ways to improve reference counting performance, both reintroduce a stop-the-world pause to reconcile reference counts and reclaim garbage objects in the zero count table.  
The immediate attraction of reference counting is that it should avoid such pauses, instead distributing the costs of memory management throughout the program.  
###### Space
 Algorithms may pay a per-object penalty, for example for reference count fields. Semispace copying collectors need additional heap space for a copy reserve; to be safe, this needs to be as large as the volume of data currently allocated, unless a fall-back mechanism is used. Non-moving collectors face the problem of fragmentation, reducing the amount of heap usable to the application.  
Systems are typically configured to use a heap anything from 30% to 200% or 300% larger than the minimum required by the program. Hertz and Berger suggest that a garbage collected heap three to six times larger than that required by explicitly managed heaps is needed to achieve comparable application performance.  
It is desirable for collectors to be not only complete but also to be prompt, that is, to reclaim all dead objects at each collection cycle. The basic tracing collectors presented in earlier chapters achieve this, but at the cost of tracing all live objects at every collection. However, modern high-performance collectors typically trade immediacy for performance, allowing some garbage to float in the heap from one collection to a subsequent one. Reference counting faces the additional problem of being incomplete; specifically, it is unable to reclaim cyclic garbage structures without recourse to tracing.
## Allocation
There are three aspects to a memory management system: (i) allocation of memory in the first place, (ii) identification of live data and (iii) reclamation for future use of memory previously allocated but currently occupied by dead objects.
##### Sequential allocation
It is simple and efficient, although Blackburn have shown that the fundamental performance difference between sequential allocation and segregated-fits free-list allocation for a Java system is on the order of 1% of total execution time.  
It appears to result in better cache locality than does free-list allocation, especially for initial allocation of objects in moving collectors.  
It may be less suitable than free-list allocation for non-moving collectors, if uncollected objects break up larger chunks of space into smaller ones, resulting in many small sequential allocation chunks as opposed to one or a small number of large ones.
![图片pic1]({{ "/assets/images/7.1.png" | absolute_url }})
##### Free-list allocation
In free-list allocation, a data structure records the location and size of free cells of memory. The algorithm searches sequentially for a cell into which the request will fit.  
When trying to satisfy an allocation request, a first-fit allocator will use the first cell it finds that can satisfy the request. If the cell is larger than required, the allocator may split the cell and return the remainder to the free-list. However, if the remainder is too small then the allocator cannot split the cell. Further, the allocator may follow a policy of not splitting unless the remainder is larger than some absolute or percentage size threshold.  
Next-fit is a variation of first-fit that starts the search for a cell of suitable size from the point in the list where the last search succeeded. When it reaches the end of the list it starts over from the beginning. The idea is to reduce the need to iterate repeatedly past the small cells at the head of the list. While next-fit is intuitively appealing, in practice it exhibits drawbacks.  
Best-fit intuitively seems good with respect to fragmentation, but it can lead to a number of quite small fragments scattered through the heap. First-fit can also lead to a large number of small fragments, which tend to cluster near the beginning of the free-list. Next-fit will tend to distribute small fragments more evenly across the heap, but that is not necessarily better. The only total solution to fragmentation is compaction or copying collection.
##### Summary
There are some particular issues to consider when designing an allocator for a garbage collected system: Allocation cannot be considered independently of the collection algorithm. In particular, non-moving collectors such as mark-sweep more or less demand a free-list approach as opposed to sequential allocation, and some local allocation buffer approaches also use sequential allocation in conjunction with mark-sweep collection. Conversely, sequential allocation makes the most sense for copying and compacting collectors, because it is fast and simple. It is not necessarily much faster than segregated-fits free-list allocation, but its simplicity may offer better overall reliability.  
If a collector uses mark-sweep but offers occasional or emergency compaction to eliminate fragmentation, then it needs to provide for updating the allocation data structures to reflect the state of the world after compaction.  
Block-based allocation can reduce per-object overheads, both for the language implementation and for collector metadata. This may be offset by the space consumed by unallocated cells and the unusable space within some blocks. Block-based allocation may also fit well with organisations that support multiple spaces with different allocation and collection techniques.  
Segregated-fits is generally faster than single free-list schemes. This is of greater importance in a garbage collected context since programs coded assuming garbage collection tend to do more allocation than ones coded using explicit freeing.
Because a collector frees objects in batches, the techniques designed for recombining free cells for explicit freeing systems are less relevant. The sweep phase of mark-sweep can rebuild a free-list efficiently from scratch. In the case of compacting collectors, in the end there is usually just one large free chunk appropriate for sequential allocation. Copying similarly frees whole semispaces without needing to free each individual cell.
## Partitioning the heap
It is often effective to split the heap into partitions, each managed under a different policy or with a different mechanism. The best known example is generational collection, which segregates objects by age and preferentially collects younger objects. These reasons include object mobility, size, lower space overheads, easier identification of object properties, improved garbage collection yield, reduced pause time, better locality, and so on.
##### Partitioning by mobility
The first is Conservative stack scanning, which treats every slot in every stack frame as a potential reference, applying tests to discard those values found that cannot be pointers . Since conservative stack scanning identifies a superset of the true pointer slots in the stack, it is not possible to change the values of any of these. Thus, conservative collection cannot move any object directly referenced by the roots. However, if appropriate information is provided for objects in the heap, then a mostly-copying collector can safely move any object except for one which appears to be directly reachable from ambiguous roots.
##### Partitioning by size
Sometimes It may also be undesirable to move some objects. A common strategy is to allocate objects larger than a certain threshold into a separate large object space (LOS). Large objects are typically placed on separate pages, and are managed by a non-moving collector such as mark-sweep. Notice that, by placing an object on its own page, it can also be 'copied' virtually, either by Baker's Treadmill or by remapping virtual memory pages.

##### Partitioning for space
It may be useful to segregate objects in order to reduce overall heap space requirements. Particularly for young objects which benefit from being laid out in allocation order.  
Both copying and sliding collectors eliminate fragmentation and allow sequential allocation. However, copying collectors require twice the address space of non-moving collectors and mark-compact collection is comparatively slow. It is therefore often useful to segregate objects so that different spaces can be managed by different memory managers.  
To those objects with higher rates of allocation and higher expected mortality can be placed in a space managed by a copying collector for fast allocation and cheap collection.
##### Partitioning by kind
Physically segregating objects of different categories also allows a particular property, such as type, to be determined simply from the address of the object rather than by retrieving the value of one of its field or, worse, by chasing a pointer. It offers a cache advantage since it removes the necessity to load a further field and segregation by property, whereby all objects sharing the same property are placed in the same contiguous chunk in order to allow a quick address-based identification of the space, allows the property to be associated with the space rather than replicated in each object's heade
##### Partitioning for yield
The best known reason for segregation is to exploit object demographics.   
If the distribution of object lifetimes is sufficiently skewed, then it is worth repeatedly collecting a subset (or subsets) of the heap rather than the entire heap. When the space chosen for collection has a sufficiently low survival rate, a partitioned collection strategy can be very effective.

##### Partitioning to reduce pause time
By restricting the size of the condemned space that the collector traces, we bound the volume of objects scavenged or marked, and hence the time required for a garbage collection. Nevertheless, in general, partitioned collection cannot reduce worst-case pause times.

##### Summary
There're many other methods: Partitioning for locality; Partitioning by thread; Partitioning by availability and Partitioning by mutability. All of them lead to a question that how to partition.  
To align chunks on power of two boundaries. In that case an object's space is encoded into the highest bits of its address and can be found by a shift or mask operation.   
The alternative is to implement spaces as discontiguous sets of chunks of address space.

Partitioning decisions can be made statically (at compile time) or dynamically — when an object is allocated, at collection time or by the mutator as it accesses objects.

## Generational garbage collection
It is told that long-lived objects tend to accumulate in the bottom of a heap managed by a mark-compact collector, and that some collectors avoid compacting this dense prefix. 
Generational collectors extend this idea by not considering the oldest objects whenever possible.It segregate objects by age into generations, typically physically distinct areas of the heap. Younger generations are collected in preference to older ones, and objects that survive long enough are promoted (or tenured) from the generation being collected to an older one.
##### Measuring time
There are two choices: bytes allocated or seconds elapsed. On the other hand, internally, object lifetimes are better measured by the number of bytes of heap space allocated between their birth and their death.  
Unfortunately, the counter will include allocation by threads unrelated to the object in question.  
In practice generational collectors often measure time in terms of how many collections an object has survived, because this is more convenient to record and requires fewer bits.
##### Generations and heap layout
The weak generational hypothesis, that most objects die young. On the other hand, there is much less evidence for the strong generational hypothesis that, even for objects that are not newly-created, younger objects will have a lower survival rate than older ones.

 Collectors may use two or more generations, which may be segregated physically or logically. Each generation may be bounded in size or the size of one space may be traded against that of another.
##### Multiple generations
Adding further generations is one solution to the dilemma of how to preserve short pause times for nursery collections without incurring excessively frequent full heap collections, because the oldest generation has filled too soon. The role of the intermediate generations is to filter out objects that have survived collection of the youngest generation but do not live much longer. If a collector promotes all live objects en masse from the youngest generation, the survivors will include the most recently allocated objects despite the expectation that most of these will die very soon. By using multiple generations, the size of the youngest generation can be kept small enough to meet expected pause time requirements without increasing the volume of objects dying in the oldest generation shortly after their promotion.
##### Adapting to program behaviour
Adaptation is needed because objects' lifetime demographics are neither random nor stationary. Instead real programs (unlike toy ones or synthetic benchmarks) tend to operate in phases. There are a wide range of common patterns of behaviour. A set of live objects may gradually accumulate and then die all at once.  
It is useful to be able to adapt garbage collectors in general to the mutator's behaviour, for example to reduce expected pause time or to improve overall throughput.
##### Summary
Generational garbage collection has proved to be a highly effective organisation, offering significant performance improvements for a wide range of applications. By limiting the size of the youngest generation, and concentrating collection effort on that generation, expected pause times can be reduced to a point where they are usually unnoticeable in many environments. This tactic can also increase overall throughput in two ways. First, it reduces the frequency with which long lived data is processed, and thereby not only reduces processing effort but also gives older objects more time to die (so that they need not be traced at all). Second, generational collectors usually allocate young objects sequentially in a nursery area. Sequential allocation obtains cache locality benefits because the memory consumption pattern is predictable.

We looked at generational and other age-based collection schemes. Those algorithms partitioned objects by their age and chose a partition to collect based on some age-related property. For example, generational collectors preferentially collect the youngest partition (or generation). Although this strategy in particular is highly effective for a wide range of applications, it does not address all the problems facing the collector. So we examine schemes outside the age-based collection framework but still based on partitioning the heap.  
There're Large object spaces; opological collectors; Bookmarking garbage collection and Ulterior reference counting.

## Run-time interface
##### Interface to allocation
For our purposes we break allocation and initialisation down into three steps, not all of which apply in every language or case.
1. Allocate a cell of the proper size and alignment. This is the job of the allocation subsystem of the memory manager.  
2. System initialisation. By this we mean the initialisation of fields that must be properly set before the object is usable in any way. For example, in object-oriented languages this might include setting the method dispatch vector in the new object. It generally also includes setting up any header fields required by either the language,the memory manager or both. For Java objects this might include space for a hash code or synchronisation information, and for Java arrays we clearly need to record their length somewhere.  
 3. Secondary initialisation. By this we mean to set (or update) fields of the new object after the new object reference has 'escaped' from the allocation subsystem and has become potentially visible to the rest of the program, other threads and so on.

## Concurrency preliminaries
A processor is a unit of hardware that executes instructions. A thread is a sequential program, that is, an execution of a piece of software. A thread can be running (also called scheduled), ready to run, or blocked awaiting some condition such as arrival of a message, completion of input/output, or for a particular time to arrive. A scheduler, which is usually an operating system component, chooses which threads to schedule onto which processors at any given time. 
##### Interconnect
What distinguishes a multiprocessor from the general case of cluster, cloud or distributed computing is that it involves shared memory, accessible to each of the processor.
## Fences and happens-before
A memory fence is an operation on a processor that prevents certain reorderings of memory accesses. In particular it can prevent certain accesses issued before the fence, or certain accesses issued after the fence, or both, from being performed in an order that places them on the other side of the fence. For example, a total read fence requires all reads before the fence to happen before all reads issued after the fence.  
This notion of happens-before can be formalised, and refers to requirements on the order in which operations occur on memory. Thus, the total read fence imposes a happens-before relationship between each previous read and each later read. Typically, atomic operations imply a total fence on all operations: every earlier read, write, and atomic operation must happen-before each later read, write, and atomic operation. However, other models are possible, such as acquire-release. In that model, an acquiring operation (think of it as being like acquiring a lock) prevents later operations from being performed before the acquire, but earlier reads and writes can happen after the acquire. A releasing operation is symmetrical: it prevents earlier operations from happening after the release, but later reads and writes may happen before the release. In short, operations outside an acquire-release pair may move inside it, but ones inside it may not move out. This is suitable for implementing critical sections.





[back](./)