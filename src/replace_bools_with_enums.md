# Replace Bools with Enums

A Rust [`enum`][std_lib_enum] is great at representing a fixed set of values. For
instance, an enum can be used to represent the state of a connection with
variants like `Ready`, `Connecting`, `Connected`, and `Disconnected`.

Even though Rust enums can be used to represent many different values, one
common usage is to only have 2 different variants. [`Result`][std_lib_result]
and [`Option`][std_lib_option] are both enums with only 2 variants.

A common data type which has only 2 states is the [`bool`][std_lib_bool] type.

Replacing bools with enums is beneficial.

## Example

### Original Code

```rust,noplayground
fn process(is_strict: bool) {
  // ...
}

process(true)
```

### Improved Code

```rust,noplayground
enum Tolerance {
  Strict,
  NotStrict,
}

fn process(tolerance: Tolerance) {
  // ...
}

process(Tolerance::Strict)
```

## Advantages

### Easier to Read and Write

Instead of having to understand what `true` or `false` means, a well named
variant can clearly document the intent.

```rust,noplayground
# fn process(_: bool) {
# }
process(true);
```

When reading code like the above example, `true` does not give any contextual
clues on what it means. For instance, it could mean to do something or to not do
something.

```rust,noplayground
fn process_1(is_strict: bool) {
  // ...
}

fn process_2(is_not_strict: bool) {
  // ...
}

process_1(true);
process_2(true);
```

Using a more descriptive enum can make the code clearer.

```rust
enum Tolerance {
  Strict,
  NotStrict,
}

fn process(tolerance: Tolerance) {
  // ...
}

process(Tolerance::Strict)
```

### Boolean blindness

If there are multiple `bool` parameters, you may run into [boolean
blindness][boolean_blindness].

```rust,noplayground
# fn process(read_from_db: bool, is_strict: bool) {
# }
process(true, false);
```

It is easy to mix the argument order and call the function incorrectly.
Likewise, reading the code may require double-checking the function
documentation.

```rust
enum Tolerance {
  Strict,
  NotStrict,
}

enum ReadFromDb {
  Allowed,
  Disallowed,
}

fn process(read_from_db: ReadFromDb, tolerance: Tolerance) {
  // ...
}

process(ReadFromDb::Allowed, Tolerance::NotStrict)
```

Using different enums, the code is easier to read compared to the multiple
`bool`s.

Furthermore, since the arguments are different types, the compiler will ensure
that the intended values are passed in the correct order.

### Refactoring with Types

Having explicit enum types makes refactoring easier.

If a function's boolean argument's meaning was changed (e.g. from `is_enabled:
bool` to `is_disabled: bool`), then every call to the function would have to be
checked. If an enum is used, the intent is expressed at the caller site and
misinterpreted.

If the order of multiple boolean arguments is changed, then every function call
site must be checked to ensure the values are passed correctly in the new order.
If different enum types are used, the compiler will verify the arguments are
passed correctly.

### Support More than 2 Variants

If the code needs to support more than 2 variants, enums can of course support
more variants. When using only `bool`s to support more variants, the number of
`bool` parameters increases which leads to more logic to detect correct and
incorrect combinations of values.

### Performant

Generally, a `bool` in Rust takes 1 byte. An enum with 2 variants also takes 1
byte, so there is not any performance loss by using an enum instead of a `bool`.

## Considerations

### match to exhaustively check all variants are handled

[`match`][ref_match] should be the dominant way to check an enum's value. By
leaning into exhaustive pattern matching, all variants are more likely to be
properly handled.

### Too Many Enum Types

If there are many `bool` parameters, the number of enum types may become
excessive.

## Alternatives

### Inlay Hints

IDEs can help with `bool` function parameters with parameter name inlay hints.
The editor could display the function call like:

```rust,ignore
fn process(read_from_db: bool, is_strict: bool) {
  // ...
}

// IDE would display the "read_from_db" and "is_strict: ".

process(read_from_db: true, is_strict: false);
```

Inlay hints can help when reading and writing code, but there are many tools
which do not support them. For instance, when reviewing code, it is likely only
the raw text is visible in diffs, PRs, etc. so having more descriptive code with
enums is better.

### Named Function Arguments

[Named function arguments](https://github.com/rust-lang/rfcs/issues/323) are not
currently supported by Rust. Named function arguments could allow more clarity
at the function calling site.

In theory, they would allow something like:

```rust,ignore
fn process(read_from_db: bool, is_strict: bool) {
  // ...
}

process(read_from_db: true, is_strict: false);

// Could also allow mixing the order potentially:

process(is_strict: false, read_from_db: true);
```

Inlay hints from an IDE already provide some of the basic functionality, but
named function arguments would allow for optional arguments and re-ordering of
arguments amongst other features.

You could pass in a struct parameter to achieve some of the named argument
functionality:

```rust,noplayground
struct ProcessArgs {
  read_from_db: bool,
  is_strict: bool,
}

fn process(args: ProcessArgs) {
  // ...
}

process(ProcessArgs { read_from_db: true, is_strict: false });
process(ProcessArgs { is_strict: false, read_from_db: true });
```

## Miscellaneous

### Bools as Enums

There is an [open issue](https://github.com/rust-lang/rfcs/issues/348) to
consider making `bool` a specialized enum in the Rust language.

### Other Languages

Haskell, Swift, and other languages with [algebraic sum
types][wiki_algebraic_sum_type] can also replace boolean type usage with their
Rust enum equivalents. Most language communities recommend considering the
technique for the same reasons outlined.

[std_lib_enum]: https://doc.rust-lang.org/std/keyword.enum.html
[std_lib_result]: https://doc.rust-lang.org/std/result/enum.Result.html
[std_lib_option]: https://doc.rust-lang.org/std/option/enum.Option.html
[std_lib_bool]: https://doc.rust-lang.org/std/primitive.bool.html
[boolean_blindness]: https://duckduckgo.com/?q=boolean+blindness&t=ffab&ia=web
[ref_match]: https://doc.rust-lang.org/reference/expressions/match-expr.html
[wiki_algebraic_sum_type]: https://en.wikipedia.org/wiki/Tagged_union
