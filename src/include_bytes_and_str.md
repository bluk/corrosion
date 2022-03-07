# include_bytes and include_str

[include_bytes!][std_lib_include_bytes] and
[include_str!][std_lib_include_str] are two macros provided in the core
library. They allow you to include data in a file as part of the compiled binary.

```rust,ignore
let file_data: &str = include_str!("my_file.json");
println!("{}", file_data);
```

They are one of the "nice" things in Rust. For instance, if you ever have a
"default" config, you can keep the data in a preferred file format and just
include the data as part of your executable. You can access the data like a
normal byte slice or string slice.

If you ever have small amounts of test data, you can include the data instead
of having to write code to load the file and read the data.

Including data in your executable makes the compiled artifacts larger, but it
removes the need to bundle the binary with additional data files. It also
removes the code to find, load, and process files.

[std_lib_include_bytes]: https://doc.rust-lang.org/std/macro.include_bytes.html
[std_lib_include_str]: https://doc.rust-lang.org/std/macro.include_str.html
