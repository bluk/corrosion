# Overloading Functions

Rust does not have traditional function overloading.

In some other langauges, the argument types and the number of arguments can lead
to unique function signatures and effectively allow the same name to be used:

```java
class User {
  void doSomething(int id) {
  }

  void doSomething(String id) {
  }

  void doSomething(int id, String name) {
  }
}
```

While all 3 methods share the same `doSomething` name, they can each do
something unique and the only thing in common could be the method name.

Rust does not allow multiple function definitions with the same name. So Rust
does not allow multiple functions with the same name to vary with the number of
arguments or the types of parameters.

Instead, Rust functions can use generics to overload a function.

## Effectively Overloading via Generics

```rust
fn doSomething<T: AsRef<str>>(s: T) {
  print!("{}", s.as_ref());
}

doSomething(String::from("Hello"));

doSomething(&" world");

let s = "!".to_string();
let s = &s;
doSomething(s);

struct MyStruct {
  inner: String,
}

impl AsRef<str> for MyStruct {
  fn as_ref(&self) -> &str {
    &self.inner
  }
}

let s = MyStruct { inner: "Hello world!".to_string() };
doSomething(&s);
```

The `doSomething` function uses generics to accept many different types. There
are many standard traits such as [`Into`][std_into], [`TryInto`][std_try_into],
[`AsRef`][std_as_ref], and [`AsMut`][std_as_mut] which are useful as generic
type parameter constraints. (Note: A type should implement [`From`][std_from]
instead of `Into`.)

## Different function names

An alternative of course is to have different names for the functions. Instead
of `doSomething`, have `doSomethingWithI64` and `doSomethingWithStr`.

Of course, the functions could actually do something completely different. A
nice property of having only one function with generic type parameters is the
function definition is the same regardless of the concrete types used (hence a
generic function).

If differently named functions only do type conversions, then it is often better
to use a generic function.

```rust
fn calculate_with_string(value: String) {
  calculate_with_str(&value)
}

fn calculate_with_str(value: &str) {
  println!("{}", value);
}
```

The above functions are a code smell when a single generic function is possible.

```rust
fn calculate<T: AsRef<str>>(value: T) {
  println!("{}", value.as_ref());
}
```

On the other hand, there are times when function implementations are not
generic. Constructors are functions which can often have different
implementations.

```rust
struct MyType {
  value: i64
}

impl MyType {
  fn with_str<T: AsRef<str>>(value: T) -> Result<Self, std::num::ParseIntError> {
    Ok(MyType { value: value.as_ref().parse()? })
  }

  fn with_i64(value: i64) -> Self {
    MyType { value }
  }
}
```

The `with_str` function is failable and requires additional processing while the
`with_i64` is simple and merely wraps the value. While overly simplistic, there
are many constructors which operate differently enough that it is easily
justifiable to have differently named functions.

[std_into]: https://doc.rust-lang.org/std/convert/trait.Into.html
[std_try_into]: https://doc.rust-lang.org/std/convert/trait.TryInto.html
[std_as_ref]: https://doc.rust-lang.org/std/convert/trait.AsRef.html
[std_as_mut]: https://doc.rust-lang.org/std/convert/trait.AsMut.html
[std_from]: https://doc.rust-lang.org/std/convert/trait.From.html
