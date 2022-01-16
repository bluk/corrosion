# Access Control

[Visibility/access scopes][ref_visibility_and_privacy] in Rust can be granular,
but often you only need to know about `pub(crate)` and `pub`.

If the library has relatively few developers or is relatively small,
encapsulation should be considered at the crate boundary versus individual
modules within the crate.

## pub(crate)

Use `pub(crate)` for access level control when you need to access something
outside of its module. It allows you to access the type/field/function/item
throughout the crate without making it available to any code outside the crate.

```rust,ignore
pub(crate) mod people {
  pub(crate) struct Person {
    pub(crate) name: String,
  }

  pub(crate) fn announce(p: Person) {
    // ...
  }
}

mod organization {
  struct Org {
    people: Vec<crate::people::Person>,
  }
}
```

If there are no invariants that need to be kept between the fields, then it is
often unnecessary to generate getters/setters.

## pub

If you are building a library, be judicious with `pub`.

Do not make individual fields of a struct/enum `pub`.

[ref_visibility_and_privacy]: https://doc.rust-lang.org/reference/visibility-and-privacy.html
