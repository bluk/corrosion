# System Language

A ["systems programming language"][wiki_systems_language] can be generally hard to
define. If a programming language can build operating systems, system
daemons, command line utilties, web browsers, compilers, interpreters, web
services, and more, then there is little the programming language cannot do.

C and C++ are both accepted as systems programming languages and are commonly
used. Even machine learning applications which are developed in other languages,
like Python, use C underneath in the interpreter and in the common numeric
implementations behind popular Python libraries.

Most people associate high performance with a systems programming language.
Other people view system programming languages as languages which allow direct
access to the physical hardware. Being able to point to a random address in
memory and read some byte is one possible feature of the language.

In my view, systems programming languages are defined by the constant reminder
of the underlying physical hardware and the operating context.

- What is the word size on the target machine?
- Should a 32-bit integer be used or is an 8-bit integer ok?
- Will memory be allocated on the heap or will the value be placed on the stack?
- When and how is memory freed?
- How many processing units are on the target machine? Should the program be
  multithreaded?
- What engine, runtime, or set of system calls should be used on a target
  machine to achieve the best efficiency?

System programming langauges can have high level abstractions such as iterators
and trees, but underneath it all, there is always the context of a machine.

So a systems programming language can do everything. The question is should it
be used for everything.

[wiki_systems_language]: https://en.wikipedia.org/wiki/System_programming_language 
