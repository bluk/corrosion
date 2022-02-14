# Newtype

Newtype is a pattern in Rust copied from other strongly typed languages. It is
a wrapper around a single data type field.

```rust
struct UserId {
  user_id: i64
}
```

But commonly, it is written like:

```rust
struct UserId(i64);
```

You can access the data field like:

```rust
# struct UserId(i64);
let uid = UserId(1001);
let _ = uid.0;
```

## Why

The pattern can be considered too simple, but there are many reasons why the
pattern is useful.

### Semantics

When passing a value around, it is helpful to have semantic meaning
associated with a type, especially with primitive values.

Take a variable which is suppose to represent a user id. It could be just an
`i64` type or it could be a `UserId` type which wraps an `i64`. By giving the
variable a stronger type, the type can be used to enforce the proper arguments
are passed to a function.

```rust
# struct UserId(i64);
# struct User;
fn get_user_1(user_id: i64) -> User {
  todo!()
}

fn get_user_2(user_id: UserId) -> User {
  todo!()
}
```

Any `i64` value could mistakenly be passed to `get_user_1`, but only `UserId`
values can be used with `get_user_2`. Perhaps it is not too important for a
single parameter function, but if there are many parameters of the same type, it
can be more beneficial.

```rust
# struct Color;
fn new_color(r: f32, g: f32, b: f32, a: f32) -> Color {
  todo!()
}
```

The `new_color` function takes 4 `f32` values but unless you are familiar with
the order, the arguments could be mixed up. Or there could be an assumption that
instead of RGBA, a CMYK color model is used.

Of course, having a unique type for every parameter may be overkill.

While there could be more code which the compiler has to evaluate, the Rust
compiler will generally remove the wrapper type used in the newtype pattern. In
other words, the `UserId` type becomes a zero-cost abstraction which should not
impose any performance or efficiency penalty compared to the original `i64`
type. So the code can use a newtype to improve program correctness by strong
type checking for "free" effectively.

### Alternatives for Argument Passing

An alternative to the multiple same type parameters is to create a new type for
the arguments like:

```rust,no_run
# struct Color;
struct ColorArgs {
  r: f32,
  g: f32,
  b: f32,
  a: f32,
}

fn new_color(args: ColorArgs) -> Color {
  todo!()
}

let _ = new_color(ColorArgs { r: 1.0, g: 1.0, b: 1.0, a: 1.0 });
```

In a way, the workaround gives names to the arguments at the call site. There
are various proposals for formal named argument parameters in Rust RFCs.

### Trait Coherence / Orphan Rule

Implementing traits on types follows several rules. The most prominent is the
[orphan rule][reference_orphan_rule]. It specifies when a trait can be
implemented for a type. The basic idea is a type can only implement a trait if
either the type or the trait was created in the current crate. It prevents
conflicting implementations of traits on types. The actual rule is more subtle
and is worth understanding (e.g. `From` implementations involving a local crate
type into a standard library type work, and yet `From` and the standard library
type belong to the standard library).

Using the newtype pattern, the orphan rule is "worked around".

Suppose a foreign type in a different crate does not implement serde's
`Serialize` trait, but you want to serialize the type's data.

```rust
// In a third party crate:
pub struct Coordinates {
  pub x: i32,
  pub y: i32,
}

// In local crate:
struct MyType1 {
  pos: Coordinates,
  radius: i32,
}

struct MyType2 {
  pos: Coordinates,
  len: i32,
}
```

You cannot add a `Serialize` implementation to `Coordinates`, but you can wrap
the type and then implement the trait like:

```rust
# pub struct Coordinates {
#   pub x: i32,
#   pub y: i32,
# }
# trait Serialize { }
// In local crate:

struct MyCoordinates(Coordinates);

impl Serialize for MyCoordinates {
  // ...
}

struct MyType1 {
  pos: MyCoordinates,
  radius: i32,
}

struct MyType2 {
  pos: MyCoordinates,
  len: i32,
}
```

The newtype pattern is one way to "work around" the orphan rule.

### Multiple Implementations for Single Trait

A trait can only be implemented once for a type. So if you have a
[`Display`][std_lib_display] implementation for a `Date` type, there can be
no other `Display` implementation.

```rust
use std::fmt;
use std::time::Instant;

struct Date(Instant);

impl fmt::Display for Date {
  fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
    todo!("Print a locale specific date")
  }
}
```

Multiple attempts at implementing a trait for a type will result in a compiler
error. Sometimes the crate which declared a trait will have blanket
implementations. In most cases, it is helpful to have blanket implementations,
but it can lead to conflicts.

If you want to have different implementations, then multiple newtypes can be
created with the original type. Each newtype can have its own trait
implementation. For instance, if there is a `Date` type with a `Display` trait
impelmentation, then perhaps in some usages, the local specific format should be
used, and in other cases, the [ISO 8601][wiki_iso_8601] format should be used.

[reference_orphan_rule]: https://doc.rust-lang.org/reference/items/implementations.html#orphan-rules
[std_lib_display]: https://doc.rust-lang.org/std/fmt/trait.Display.html
[wiki_iso_8601]: https://en.wikipedia.org/wiki/ISO_8601
