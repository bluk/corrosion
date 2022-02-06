# Basically Just the Data

In data oriented programming, one of the underlying principles is to leave data
in a basic form. Beyond separating code (e.g. methods) from data, the idea is to
use fundamental data structures like maps, lists (vectors), strings, and
primitive types instead of custom types.

## Serialization/Deserialization

Using standard types allows easier manipulation of the data. Serialization and
deserialization is a common operation performed on data and is much easier with
the standard types.

### Output

In most languages, a JSON serializer would always know how to serialize a
language's standard collection types. When using a custom type, more complexity
is introduced into the code base. In some languages, reflection is used with
annotations. In other languages, a trait or a custom serialization method must
be implemented.

Take a Rust data structure. When using the [Serde][serde_rs] library, the
[`Serialize`][serde_serialize] trait must be implemented. While the
implementation code is probably straightforward, having to add additional code
is not desirable. Beyond testing functional and performance behavior, the
additional code requires maintenance if the data structure is ever changed.

Fortunately, Serde provides a derive macro which can generate the serialization
code. Add the `#[derive(Serialize)]` to a Rust type, and things should work. Of
course, the assumption is that all of the type's field members already implement
`Serialize`. If a data structure has a field which does not implement
`Serialize`, then `#[derive(Serialize)]` may need to be "recursively" added to
types or custom `Serialize` implementations may need to be written.

Again, if the types ever change, then there may be more undesirable maintenance.
In my experience, input and output changes as APIs evolve. Fields are eventually
added which requires changing the types and which may require fiddling with the
serialization code again.

Meanwhile, `Serialize` is already implemented for many of the Rust standard
library types including `HashMap` and `String`. If a field is added, a string
key and value can simply be inserted into the map, and there should be no worry
that the serialization code is broken.

### Input

One of the chief benefits of using generic data types is that all of the input
can be represented and kept. If a JSON object is used as input, there could be
many "unknown" keys. For some deserialization libraries, when using a custom
type, the extra data is ignored. In stricter libraries, the input with unknown
fields is rejected.

Sending a large JSON object full of unknown properties will require additional
processing time and memory. Depending on the security model, the wasted
resources can be a concern and appropriate safeguards must be enforced. For
example, if most input to an API endpoint is less than 100KB, then input which
is greater than 1MB is automatically discarded. The only unique concern to
generic data types is the retention of all of the input data in memory.

Another possible concern is the extra data values themselves can also be
malicious. Imagine a function which conditionally adds internal data to input
before passing the input along to other functions. The original input could have
already been injected with the private data with the intent to exploit a
vulnerability in the program. Custom data types without the internal data fields
would ignore or reject the malicious input, but generic data types would blindly
keep the input data. In the end though, input data should not be automatically
trusted. Separating the external input data from internal data is important
regardless of whether the data is deserialized into a custom type or a generic
type.

As an aside, custom deserialization code is a greater security concern.
Deserializing into a map, list, or a string should be straightforward and is
more likely a well-tested path. Unique types with "tricky" deserialization code
is a common source of vulnerabilities. Furthermore, it is often difficult to
connect a "simple" deserialize input function call to a deeply nested input
field's custom deserialization method.

With all that said, in most cases, extra data is harmless. Perhaps a field was
deprecated, but old clients are still sending the data. Perhaps there is a new
field which is required by a downstream function/program/service.

While the [robustness principle][wiki_robustness_principle] can be adhered to by
just ignoring irrelevant data, generic data types also allow the program to
accept all of the input. As programs evolve, it is often easier to keep all of
the input and allow individual functions to access the data as they see fit
because inputs may change.

Specifically for Rust, Serde provides a [`Value`][serde_json_value] type which
is a general data type to represent JSON values. By deserializing to `Value`,
all of the input is represented.

[serde_rs]: https://serde.rs
[serde_serialize]: https://docs.serde.rs/serde/ser/trait.Serialize.html
[wiki_robustness_principle]: https://en.wikipedia.org/wiki/Robustness_principle
[serde_json_value]: https://docs.serde.rs/serde_json/value/enum.Value.html
