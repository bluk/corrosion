# Factory Constructors

Rust does not require types to have constructors, and there is no special syntax
or naming given to constructors.

The [Rust API Guidelines][rust_api_guidelines_constructor] give a concise
summary of recommendations, but I wanted to highlight a few points.

## Factory Functions

Types can be instantiated entirely by fields:

```rust
struct Vehicle {
  name: String,
  wheel_count: u8,
}

let v = Vehicle {
  name: "Old Faithful".to_string(),
  wheel_count: 4,
};
```

If the fields of a type are not visible (e.g. a type with a private field in a
different module/crate), then the type may not be constructed directly. A
function must be provided which can construct an instance of the type.

Usually, the function is a static function associated with the type in an `impl`
block like so:

```rust
# struct Vehicle {
#   name: String,
#   wheel_count: u8
# }
impl Vehicle {
  pub fn new(name: String, wheel_count: u8) -> Vehicle {
    Vehicle {
      name,
      wheel_count,
    }
  }
}

let v = Vehicle::new("Old Faithful".to_string(), 4);
```

Note that the `new` function is not special. It is only by convention that a
Rust constructor be called `new`. It could have been written like:

```rust
# struct Vehicle {
#   name: String,
#   wheel_count: u8
# }
impl Vehicle {
  pub fn with_name_and_wheel_count(name: String, wheel_count: u8) -> Vehicle {
    Vehicle {
      name,
      wheel_count,
    }
  }
}

let v = Vehicle::with_name_and_wheel_count("Old Faithful".to_string(), 4);
```

The function could have also been a free function like:

```rust
# struct Vehicle {
#   name: String,
#   wheel_count: u8
# }
pub fn new_vehicle(name: String, wheel_count: u8) -> Vehicle {
  Vehicle {
    name,
    wheel_count,
  }
}

let v = new_vehicle("Old Faithful".to_string(), 4);
```

Constructors are closer to factory methods from other languages because there is
no special syntax, no special method name (e.g. do not need to use the type's
name like Java or `init` like Swift), and no unique privileges given to
constructing functions in Rust.

Constructors are important and common in Rust, but there is not much more to
learn compared to regular functions.

### Use of Self

`Self` with an uppercase`S` refers to the current `impl`'s type. I recommend
using `Self` whenever possible.

```rust
# struct Vehicle {
#   name: String,
#   wheel_count: u8
# }
impl Vehicle {
  pub fn new(name: String, wheel_count: u8) -> Self {
    Self {
      name,
      wheel_count,
    }
  }
}

let v = Vehicle::new("Old Faithful".to_string(), 4);
```

In the above, `Self` replaced the return type for `new(...)`. `Self` also
replaced the type in the method body, so it becamse `Self { name, wheel_count }`
instead of `Vehicle { name, wheel_count }`.

Using `Self` as a return type gives a signal that the function might be a
constructor. Functions normally do not return the implementing type except when
using the `Builder` design pattern.

Another benefit is you do not have to change the return type if the type needs to
be renamed. IDEs should handle any changes when renaming types, but in case you
are not using one.

`Self` is not solely for constructors, but it is commonly used in constructors.

## Default before empty new

When implementing an empty no argument constructor, you should always consider
implementing the [`Default`][std_default_trait] trait.

If all of the fields in the type implement the `Default` trait, then `Default`
implementation can be derived for the type:

```rust
#[derive(Default)]
struct Coordinates {
  x: i32,
  y: i32,
}

#[derive(Default)]
struct Entity {
  coords: Coordinates,
  name: String,
}

let c = Coordinates::default();
let e1 = Entity::default();

let e2 = Entity {
  name: "Test".to_string(),
  ..Default::default()
};
```

The `..Default::default()` syntax is weird, but it initializes the rest of the
struct with the default values.

There are methods which use
the `Default` trait such as
[`Option::unwrap_or_default()`][std_lib_option_unwrap_or_default] or `HashMap`'s
[`Entry::or_default()`][std_lib_entry_or_default].

`Default` can make a type easier to use in the Rust ecosystem.

## Implement From/TryFrom before conversion constructors

If there is a straightforward implementation of [`From`][std_lib_from] or
[`TryFrom`][std_lib_tryfrom] for a type, implement the `From`/`TryFrom` traits
before implementing a constructor.

I have never regretted implementing a `From` trait. By implementing the `From`
trait, the [`Into`][std_lib_into] trait is also implemented. I find calling
`MyType::from(<other type>)` and `instance_of_my_type.into()` to be natural to
convert between types. In other languages, constructors would be overloaded with
different parameter types.

Be sure to follow the traits' documentation. `From` should only be implemented
when the function can be called without panicking (e.g. no `unwrap`s).

The only time when you may want to forgo a `From` implementation is when there
can be some confusion what the value means. For instance, if a buffer of bytes
can be interpreted differently (e.g. big endian vs. little endian), then it is
better to only have explictly named constructor functions like `from_be_bytes`.

## Use Generic Functions If Multiple Similar Types

If you have multiple constructing functions like `with_string` and `with_str`
which uses the value similarly, try to use a generic function instead.

```
struct Data {
  s: String,
}

impl Data {
  fn from_string<S: ToString>(s: S) -> Self {
    Self {
      s: s.to_string(),
    }
  }
}
```

Using generics gives you overloaded (by type) functions. In addition for the
above example, I would also implement `From<String>` and `From<&str>`.

## Constructor Names

Defer to the [Rust API Guidelines][rust_api_guidelines_constructor] for naming.
`new`, `from_...`, and `with_...` are common constructor names.

Note the importance of communicating how to interpret ambiguous values (e.g. a
random byte buffer) by having descriptive function names (e.g. `from_utf8` vs
`from_utf16`).

[rust_api_guidelines_constructor]: https://rust-lang.github.io/api-guidelines/predictability.html?highlight=constructor#constructors-are-static-inherent-methods-c-ctor
[std_default_trait]: https://doc.rust-lang.org/std/default/trait.Default.html
[std_lib_option_unwrap_or_default]: https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or_default
[std_lib_entry_or_default]: https://doc.rust-lang.org/std/collections/hash_map/enum.Entry.html#method.or_default
[std_lib_from]: https://doc.rust-lang.org/std/convert/trait.From.html
[std_lib_tryfrom]: https://doc.rust-lang.org/std/convert/trait.TryFrom.html
[std_lib_into]: https://doc.rust-lang.org/std/convert/trait.Into.html
