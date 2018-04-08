---
path: "/learnings/java_memory"
title: "Learnings: Java: Memory"
---

# <<Learning_Java_Memory>>

## <<Learning_Java_Memory_Hotspot>>


### <<Learning_Java_Memory_Hotspot_Object_Layout>>

Java objects at runtime are pointers to:

  * 2 words: mark word (pointer to instance specific metadata); klass word (class wide metadata)

^^^ note with `-XX:UseCompressedOops` on this "word" may not be word sized.
^^^^ DEFAULT FOR Java 7=<

Java =< 7:

klassWord points to `PermGem` space in Java heap to hold class metadata. These were Java objects, because they live in Java header (thus require java object header).

Java >= 8:

klassWord points to memory **outside the Java heap but inside the heap of the application process**. Do not require a java object header.

#### - [REVIEW]: So if you're cookie-cuttering based purely on Java heap `-Xmx` settings, you're underestimating the actual memory required by your process? (in addition to missing the overhead of ie the `cdata` section of Unix process memory ????
s

demarked so because `klassWord` does _not_ point to an Class instance, but a struct in C memory.

### <<Learning_Java_Hotspot_Memory_Zones>>

Hypothesis: most objects live and are discarded young.

Eden -> Survivor ("S1", "S2" )-> Tenured

Uses "card table" to keep track of old objects pointing at new object

_NOTE:_ G1, not this old and young memory zones, default in Java 9+.


#### - [BOOKQUOTE]: 

> A typical pause time for a young collection on a 2G heap (with default sizing) on modern heap might well be just a few milliseconds, very frequently under 10ms.

- Optimizing Java


#### <<Learning_Java_Hotspot_Memory_Zones_And_Threads>>

pre-allocated eden space from heap divvyed up for each app thread: called thread local allocation buffers (TLABs).

(TLABs can grow if a thread needs it).


##### - [BOOKQUOTE]: 

> veral techniques to minimize space wastage due to the use of TLABs are employed. For example, TLABs are sized by the allocator to waste less than 1% of Eden, on average. The combination of the use of TLABs and linear allocations using the bump-the-pointer technique enables each allocation to be efficient, only requiring around 10 native instructions.

- Memory Management in Hotspot Virtual Machine

#### See also:

  * [Sun: Memory Management in Hotspot Virtual Machine](http://www.oracle.com/technetwork/java/javase/memorymanagement-whitepaper-150215.pdf)
  
  
### <<Learning_Java_Hotspot_Garbage_Collector_Type>>

#### Concurrent Mark and Sweep (CMS):

Phases:

  1. initial mark (stop the world)
  2. concurrent marking phase
  3. Concurrent Preclean
  3. remark (Stop the World)
  4. concurrent sweep 
  6. Concurrent Reset

*Only collector that is not compacting*. Memory fragmentation problems, requiring time by CMS allocator to plan for / break or join blocks to deal with (See: Sun: Memory Management in Hotspot Virtual Machine)
(means needs free lists)

**ONLY** available for old generations. ( Usually paired with `ParNew` or `ParallelGC` ).

Source: Memory Management in Hotspot

See also:

  * Tri-Color Invariant garbage collection; “On-the-Fly Garbage Collection: An exercise in Cooperation” (1978) by Dijkstra and (Leslie) Lamport. ALSO NOTE: initial publication contained some bugs that Dijkstra and Lamport semi independently solved slightly later. (See [Lamport's homepage](http://lamport.azurewebsites.net/pubs/pubs.html) for more information here)
  * `ParallelOld` <-- what CMS falls back on if Concurrent Mode Failure happens aka the running app collects too much garbage while in the middle of concurrently collecting garbage.

#### G1: Garbage First  <<Learning_Java_Hotspot_Java_9_Default_GC>>

_not recommended for JVMs < 1.8u40_.

**DEFAULT GARBAGE COLLECTOR IN JAVA 9**

Works on regions: 1MB-ish a piece, and 2048-4095 regions in memory at a time.

Large objects special cased into humongous region

Still has:

  * eden vs tenured regions
  * TLABs

Steps:

  1. Initial Mark (Stop the World)
  2. Concurrent Root Scane
  3. Concurrent Mark
  4. Remark (Stop the World)
  5. Cleanup (Stop the World)

G1 based around pause goals.

#### <<Learning_Java_Hotspot_Memory_When_GC_Triggered>>

  * If TLAB is full but thread needs to have yet more memory
  *