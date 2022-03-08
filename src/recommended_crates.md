# Recommended Crates

A set of recommended crates. They are not in any particular order.

* [rand](https://github.com/rust-random/rand)

  Random value generation.

* [uuid](https://github.com/uuid-rs/uuid)

  Uuid values.

* [Rust Crypto](https://github.com/RustCrypto)

  Rust implementations of common crypto algorithms. RSA/ECDSA/SHA/etc.
  Probably the most confusing thing is that most of the useful operations are
  available as traits which makes them difficult to discover.

  There are usually ways to import JWKs, PEMs/P8s, etc. which make it easier to
  work with.

* [ring](https://github.com/briansmith/ring)

  Ring implements common crypto algorithms in Rust, C, and assembly based on
  BoringSSL. More limited crypto operations and somewhat difficult to use. Keys
  must be converted to a set of supported formats which can be troulbesome.

* [serde](https://github.com/serde-rs/serde)

  Serde is the standard serialization/deserialization library for many common
  formats like JSON and YAML.

* [url](https://github.com/servo/rust-url)

  Parses URLs.
