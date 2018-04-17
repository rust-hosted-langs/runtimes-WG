# Inko

Project repository: https://gitlab.com/yorickpeterse/inko/

For [Inko](https://gitlab.com/yorickpeterse/inko):

> Does this implement a custom allocator or rely on native allocation?

Both. That is, it uses Rust's built-in allocator but managed objects (= those allocated by the language) use a custom allocator based on [Immix](http://www.cs.utexas.edu/users/speedway/DaCapo/papers/immix-pldi-2008.pdf). The code for this resides in https://gitlab.com/yorickpeterse/inko/tree/master/vm/src/immix.

> what allocation strategy?

It bump allocates into 32KB blocks, which are basically arenas.

> what do internal data structures look like? (tagged pointers? object headers?)

The allocator hands out pointers to structures on the heap. These pointers may be tagged in certain cases. Objects don't have headers, but blocks do (which are used for embedding some other data). 

> does the allocator, or a layer above it, prevent pointers outliving the lifetime of the allocator? how?

No.

> what collection strategy is implemented?

Inko uses a parallel, generational garbage collector. Each lightweight process is collected separately, thus there is no global "stop the world" phase. A process that is collected _is_ suspended for the duration of the collection.

For some Rust structures (e.g. a Rust `String` or `Vec`) we rely on Rust's `Drop` trait, and the collector will manually drop these objects when they go out of scope. Strings are stored as `Arc<String>` so they can be cheaply shared between processes.

> Does the memory management allow for concurrent allocation and gc?

While different processes can run while others are being collected, the process being collected can not. Technically Immix can support concurrent operations, though it gets rather tricky since it's a moving collector. I decided to just not support this in favour of faster GC throughput.

> green threads? OS threads?

Both. The VM has two thread pools for running processes (green threads pretty much): one for regular processes, and one for slow/blocking ones (e.g. processes performing IO operations). The language allows you to easily move a process from one pool to another (and back). 

> what mechanisms ensure heap consistency?

There's a global heap that uses synchronisation for writes, but reads are not synchronised due to the significant overhead this would access. This is OK in Inko since code won't (or at least shouldn't) write to the global heap from any process other than the first one (and Inko will soon assert that this is actually the case). Process-local heaps have no form of synchronisation since they are never accessed concurrently. Inter-process communication happens by message passing. The way this works is basically as follows:

1. Process 1 deep copies object A into a special "mailbox heap" from process 2, producing object B
2. When process 2 receives a message it will deep copy any messages stored in the mailbox heap into the process-local heap.

Access to the mailbox heap _is_ synchronised. Using the above approach we need to copy data around a few times, but it ensures that the stack/heap of a process _always_ refers to either a process-local or permanent object; never to a mailbox object. Both the mailbox and process-local heaps are collected separately and use different collection thresholds.

> What is elegant and what feels awkward in Rust?

From the top of my head:

* The `alloc` crate is (still) not stable, requiring nightly Rust. This will hurt adoption since it makes it harder to compile the code (for many distributions this means not being able to use the packaged Rust version).
* Intrinsics require a nightly compiler as well, which is annoying for the same reasons as outlined above. Inko uses these to speed up garbage collection (though you can opt-out if desired)
* `Arc` has additional overhead to support weak references. Inko doesn't need these and I ended up building my own `Arc` (https://gitlab.com/yorickpeterse/inko/blob/master/vm/src/arc_without_weak.rs). This structure ended up performing quite a bit better when dropping many reference counted Rust structures.
* Pretty much the only reliable way of writing an interpreter loop is to use `match` as there is no computed goto. The alternatives are relying on TCO (which isn't always available/enabled), or inline assembly (not portable, pain to maintain, etc). This means most interpreters will have more overhead compared to those written in e.g. C/C++ (ignoring any optimisations certain CPUs may provide). 

> how do data structures in the hosted language interact with the Rust language?

One of the goals for Inko was to write as much in Inko itself. This means that the only Rust structures we expose are simple ones such as `String` and `Vec`. All of these are exposed in such a way that as few Rust internals leak into Inko. More complex structures such as [HashMap](https://gitlab.com/yorickpeterse/inko/blob/master/runtime/std/hash_map.inko) are implemented in Inko itself and typically use a few primitive instructions (e.g. there's an instruction for hashing integers/floats/etc).

Rust structures are wrapped using a VM structure called `Object`. The `Object` structure has 3 fields (with a total size of exactly 32 bytes):

1. A pointer to the prototype
2. A pointer to a Rust HashMap used for storing attributes
3. An enum containing the value (a String, Vec, etc)

When an object goes out of scope the corresponding enum (called an "object value" in the VM) is also dropped by a finaliser thread.

> is mutable aliasing prevented in any way? compile-time or run-time?

No. On Rust level there's nothing stopping 2 threads from obtaining a mutable reference to the same object. In practise this won't/can't happen because memory is isolated per process.

> Garbage collection posts and papers to review

I would definitely add [Immix](http://www.cs.utexas.edu/users/speedway/DaCapo/papers/immix-pldi-2008.pdf) to this list since it's a very good garbage collection/allocator system.
