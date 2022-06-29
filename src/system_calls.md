# System Calls

System calls are what make a program interesting to the machine.

A machine usually has an operating system or some other software managing the
hardware which consists of a kernel. The kernel provides functions which are
known as system calls. System calls provide services and functionality which is
abstracted over different hardware.

Input/output (I/O), such as using a disk or network, is accomplished via system
calls. Memory allocation is perhaps the most infamous system call. (Notably, the
`malloc` function itself is not usually a system call but it does make system
calls in its implementation.)

In most environments, system calls are not directly made from a program. Whether
it be through a programming language's standard library, the system provided
`libc`, some other dependency, or some combination of dependencies, system calls
are usually made through many layers of code. Kernels may not guarantee that
direct calls to the kernel are ABI stable, so programs are required to
indirectly go through a library which provides functions which do guarantee ABI
stability.

In many cases, system calls take many orders of mangitude longer to execute than
non-system calls. Adding two integers can be done within one CPU clock cycle
which is typically within a nanosecond. Reading from a disk can take
microseconds (100,000 ns) on a SSD or even milliseconds (1,000,000 ns) on a HDD.
The majority of the time in a system call is spent waiting for the hardware to
respond, but there is overhead when switching from application code
execution to executing a system call.

System calls are required for practically all programs. For instance, if there
are no system calls, the program would not be able to output any results.

However, system calls should not be made needlessly. If a synchronous `blocking`
system call is made, the program will stop execution on the thread and wait for
the result. Therefore, for single threaded programs, the entire program is stopped.

Some system calls can be more efficiently called. Instead of writing a single
byte or 100 bytes at a time, entire pages of bytes (1K, 4K, etc.) can be
written in a single system call. Most programming languages' standard libraries
provide a "buffered" writer. Write to the buffer and when the buffer is big
enough, the underlying system call is made to do the real system call write.
Buffered writing comes with a few tradeoffs such as the memory allocated for the
buffer, some very remote possibility that a delayed write will cause an issue,
and programming errors if the write buffer is not "flushed" to force a write
when there are no more bytes to be written and there are still bytes left in the
buffer.

Instead of just waiting for the result of a system call, a program can try
various methods to productively use the CPU. Multi-threaded programs can stall
on one thread but another thread can continue executing. There is some time
spent switching between threads, but it is usually better to switch threads than
to stop execution entirely. "Asynchronous" runtimes can be used which switches
to a different task (similiar to multi-threaded programs).

System calls are the bridge from pure computation to hardware.
