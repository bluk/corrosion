# Memory Management

Memory management is perhaps the most opinionated aspect of a programming
language. It is fundamental and affects everything from what code is written,
the performance of the program, and how code interacts with one another (think
how is memory managed between libraries and which code is responsible for
allocating and freeing memory).

## Memory Leaks

Leaking memory has many connotations but the one I will use is not freeing
memory after its last use. For example, memory can be leaked because code to
free the memory was not called. If a programming language is garbage collected
and the garbage collector does not handle cyclical references, memory can also
be leaked.

Rust does not prevent memory leaks. It is not considered a safety issue. There
are even safe Rust methods like `Box::leak` which intentionally leak memory.
Rust does manage memory by freeing memory after use (usually via a `Drop`
implementation), but Rust does not guarantee that memory cannot be leaked. It's
a subtle yet important distinction. In other words, in the vast majority of Rust
programs, memory is managed so that all allocated memory is freed after its last
use; however, if code is called which does leak memory, it is not considered an
error or safety issue by the Rust toolchain.

Memory leaks are not necessarily bad. For instance, after a program ends, most
environments will clean up memory allocations. Memory leaks do not lead to
errors, but memory exhaustion does. If a program asks to allocate more memory
than an environment has, then an out of memory error occurs and most programs do
not know how to handle the error.

Sometimes memory is allocated with no intention to be used, and it is expected
behavior. For example, imagine a garbage collector which runs after some
percentage of memory is allocated relative to the existing memory used after its
last garbage collection run. Imagine a long lived program which handles network
requests. When the program is not handling requests, the program may use a
relatively tiny amount of memory. When it handles requests, the memory usage
grows relatively big. The garbage collector could be constantly triggered as the
number of requests changes the amount of memory used. In order to stop the
frequent garbage collection, a program could allocate a relatively huge amount
of memory upfront so that the garbage collector would only run after a huge
number of requests is processed. The "huge amount of memory" is effectively
unused in a traditional sense.

On the other hand, leaking memory is not generally encouraged nor desirable. A
program could be running in an environment where memory is very limited; if some
code allocates but does not free memory, then it is more likely that an out of
memory will occur.

## Memory Fragmentation

While memory leaks are given a bad reputation, memory fragmentation is a problem
lurking in the background of many programs. When memory is allocated, a
memory allocator needs to find a sequence of memory to reserve for the
allocation.

Imagine a computer only has 8KiB of memory. A program could request to allocate
1KiB, then 2KiB, then another 1Kib. The memory allocator allocates the requests
next to each other in memory. Then the program could free the 2Kib block.
Finally, the program requests to allocate another 1KiB. There is 2KiB free
in-between the two still reserved 1KiB blocks. Should the memory allocator
re-use the 2KiB freed block or should the allocator keep reserving more memory
at the tail end of the existing allocations? In either case, there will be a gap
of free memory between allocated memory.

In most programs, allocating and freeing memory can seemingly happen randomly as
different code paths are taken. There will be many gaps of memory which
fragments the memory.

There are two issues with memory fragmentation. In theory, the memory allocator
could run into a situation where it cannot find any free memory due to
fragmentation. For instance, if a byte was allocated every 1KiB, and then an
allocation was made for 1KiB of memory, the memory allocator would not be able
to find 1KiB of contiguous memory. The problem could occur in limited memory
environments.

More commonly, memory fragmentation can cause performance issues. Fetching from
main memory takes a substantial amount of time compared to reading from CPU
L1/L2/L3 caches. Spatial locality is less likely to be achieved as values are
scattered with un-useful gaps of free memory between them. The longer a program
runs, the more likely that memory fragmentation becomes an issue.

There are a few ways to avoid memory fragmentation.

1. A compacting garbage collector will move allocated memory around to put all
   of used memory adjacent to one another. The compaction step helps improve
   spatial locality as well as makes it easier for the garbage collector
   algorithm in some cases.
2. Avoid allocating memory. Referencing (a.k.a. borrowing) existing data (e.g.
   zero copy deserialization) is one method. Another way is to re-used existing
   allocated memory. For instance, if a `Vec` removes an element, the `Vec` is
   not moved and re-allocated to a different location in memory. The capacity
   remains the same and elements are moved within the existing memory
   allocation.

## Use After Free

The biggest issue in memory management is the use after free problem. Programs
experience undefined behavior when memory is used after being freed. Any value
could potentially exist in freed memory so code could execute in any number of
undefined ways. The program could crash (e.g. `SEGFAULT`) or it could be
exploited.

Rust guarantees that memory will never be used after being freed. The compiler
ensures that safe Rust code follows rules which ensure memory is never used
after being potentially freed.

Safe Rust is built on top of unsafe code. Unsafe code may have bugs which cause
use after free. While it may sound risky, it is not any different than any
garbage collected language. Under all the layers of abstraction, there is some
code which has to manage memory. The theory is that if the surface area for
possible bugs is greatly reduced, then the program's safety is greatly
increased. In reality, there are far fewer memory related issues in memory safe
languages.

Using memory safe languages is the best way to stop use after free bugs.
Traditionally memory safe languages are associated with some sort of garbage
collection, but Rust is a language which does not use garbage collection and is
memory safe.

In memory unsafe code, usually there are tools such as memory sanitiziers which
can detect use after free issues (amongst many other memory related bugs).

## Garbage Collection

Garbage collection gets a bad reptutation due to the out of band garbage
collection process. In most cases, garbage collection can lead to latency or
pauses during a program's execution; however, the potential latency/pauses are
generally acceptable.

Garbage collection can take various forms with many algorithms. The Java Virtual
Machine has perhaps the most well known garbage collection algorithms and it is
considered one of the best implementations. Throughout the years, there have
been tracing, generational, and many more algorithms and implementations with
various runtime tweaks that can be made.

For most programs, garbage collection is perfectly acceptable even if there are
performance issues (see Python and Ruby), because it allows programmers to be
more productive without worrying about memory and the performance difference is
not important.

Perhaps the biggest issue with garbage collection is whether or not garbage
collected languages are deployable to the desired environment. For instance,
WebAssembly does not currently support garbage collection so any code written in
a garbage collected language has a difficult time getting compiled to Wasm. More
practically, garbage collection can be an impediemnt in lower resource devices
such as embedded chips. Not only is there a performance impact but even getting
the garbage collector runtime working in a low resource environment may be
difficult.

## Reference Counting

Instead of having a garbage collector use an algorithm to determine if some data
is reachable, there is usually an atomic counter which is incremented and
decremented every time a piece of code wants to retain or release some memory.
When the reference count reaches zero, the memory is freed.

Reference counting is technically a form a of garbage collection. Python and
Objective-C/Swift use reference counting. Like other garbage collection methods,
it can be very efficient with minimal costs. However, there is still a cost
which is not always trivial.

Reference counting is available in C++ and Rust as well and is used in
situations where there may be multiple owners of some memory. Rust has two
types: `Rc` (reference counted) and `Arc` (atomic reference counted). The
difference is that `Arc` wrapped types can be sent to other threads while `Rc`
wrapped types must live on the same thread.

## Data Races

Data races are perhaps the biggest issue that Rust solves. When there are
multiple threads trying to read and write to a piece of memory, the value read
from memory may be undefined or at least not trivially determined. For instance,
data may not be written to memory all at once so reading a value may result in
half of the old value and half of the new value.

Rust mostly solves data races by establishing rules around when data can be
written to and when data can be read through its borrow checker. For safe Rust
code, data races are basically a solved problem.

Removing data races is perhaps one of the greatest code quality improvements.
Data races are "silent" bugs which do not manifest in terms of crashes usually,
so data races are difficult to detect and reproduce.

There are a few sanitiziers and other tools which can help.

## Interacting with third party code

Above all, memory management is non-trivial when interacting with third party
code. Whereas you might have a model of how code should manage memory, a third
party can have a different idea so now you have to make sure you are in
agreement and working with the third party code correctly.

Imagine you want to use a third party crate for a binary tree. When data is
passed to the tree, who owns the data? Who can read the data? When is it
possible to change the data?

Memory managed languages are not perfect with interacting with third party code.
For instance, a Java `java.util.HashSet` has a strict requirement that any
object added to the set must not change its `hashCode()` value while the object
is part of the set. So if you add a `Item` to a `HashSet` and then modify one of
its properties like change a color from blue to red, the `Item`'s `hashCode()`
value should not change. However, most Java `hashCode()` hashes over all of an
object's properties to produce its `hashCode()` value. In other words, memory
safety did not prevent violating a condition of using the code.

Rust provides memory ownership rules which not only help memory management
but also does provide some more rules for enforcing additional invariants.
