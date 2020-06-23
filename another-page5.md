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























[back](./)