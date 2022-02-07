# self in fn Definitions

In many languages, `this` is a magical language keyword. You have a method on a
type like:

```java
class User {
  int id;

  /* ... */
  int getID() {
    return this.id;
  }
}
```

Where did `this` come from? Of course, it's automatically provided by the
language. In practically all cases, instance methods are similar to free functions
with the current instance being passed as the first parameter.

In other words, the compiler actually generates code like:

```java
class User {
  int id;

  /* ... */
  static int getID(User this) {
    return this.id;
  }
}
```

So, when you originally called:

```java
User user = new User();
user.getId();
```

The compiler was really generating:

```java
User user = new User();
User.getId(user);
```

There are a few languages, notably Python, which do require a `this` (usually
named `self` in Python) parameter in instance method declarations, but most
languages do not require it. The omission of forcing every instance function to
have a `this` parameter could be considered syntactical sugar.

## Rust

In Rust, the [`self`][std_lib_self] parameter is not hidden away and is required. It is an
important part of the function declaration because how `self` is declared
indicates if the value is borrowed, mutably borrowed, or owned.

```rust
# struct Database {}
struct User {
  id: i64,
}

impl User {
  fn get_id(&self) -> i64 {
    self.id
  }

  fn set_id(&mut self, id: i64) {
    self.id = id;
  }

  fn delete(self, db: Database) {
    todo!()
  }
}
```

The `self` parameter's type is indicated by the `impl` block definition.

So the code which is generated is closer to free functions like:

```rust
# struct User {
#   id: i64,
# }
fn get_id(user: &User) -> i64 {
  user.id
}
```

Furthermore, the `impl` block is important for declaring other constraints on
the functions. If you had a wrapper type like:

```rust
struct Wrapper<T> {
  inner: T
}

impl<T> Wrapper<T> {
  fn to_inner(self) -> T {
    self.inner
  }
}
```

The generated code is similar to:

```
# struct Wrapper<T> {
#   inner: T
# }
fn to_inner<T>(wrapper: Wrapper<T>) -> T {
  wrapper.inner
}
```

Declaring methods on a type is not too magical. In the end, methods are merely
conveniences and standalone free functions could be used. In Rust, additional
information is conveyed with the `self` parameter related to the borrow rules.

[std_lib_self]: https://doc.rust-lang.org/std/keyword.self.html
