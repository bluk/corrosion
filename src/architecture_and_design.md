# Architecture and Design

Rust [empowers][rust_book_empowerment] developers to write safe and efficient
code. The language, standard library, and most of the Rust ecosystem focuses on
performance almost as much as memory safety.

However, Rust only empowers. Not all Rust code is fast. Not all Rust code uses
the least amount of memory possible. And yes, there is some (relatively tiny)
unsafe code which may have undefined behavior or other issues. Testing is
required to ensure that code works as intended.

Rust enables and guides developers to more easily write the best code possible,
but the underlying problems (e.g. data locality) are similar to concerns which
other programming language ecosystems have.

Still, Rust has created some new paradiagms and idioms to address correctness
and performance issues. Even if not specific to Rust itself, there are several
existing architectures and designs which are more suitable for Rust.

[rust_book_empowerment]: https://doc.rust-lang.org/stable/book/foreword.html
