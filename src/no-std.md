# No-std support

The `serde` crate has a Cargo feature named `"std"` that is enabled by default.
In order to use Serde in a no\_std context this feature needs to be disabled.
Modify your Serde dependency in Cargo.toml to opt out of enabled-by-default
features.

```toml
[dependencies]
serde = { version = "1.0", default-features = false }
```

Be aware that Cargo features are unioned together across your entire dependency
graph. That means if any other crate you depend on has not opted out of Serde's
default features, you will build Serde with the std feature enabled whether or
not your direct dependency on Serde has `default-features = false`.

In particular, a dependency on `serde_json` always needs Serde built with std.
If you need JSON support without a standard library, please use
[`serde-json-core`] instead of `serde_json`.

[`serde-json-core`]: https://crates.io/crates/serde-json-core

### Derive

The `#[derive(Serialize, Deserialize)]` [derive macros] work just the same in a
no-std crate.

```toml
[dependencies]
serde = { version = "1.0", default-features = false, features = ["derive"] }
```

Some deserialization features that require a heap-allocated temporary buffer
will not be available in no-std mode without a memory allocator. In particular
[untagged enums] cannot be deserialized.

[derive macros]: derive.md
[untagged enums]: enum-representations.md

### Memory allocation

Opting out of the `"std"` feature of Serde removes support for any standard
library data structures that involve heap memory allocation, including `String`
and `Vec<T>`. It also removes some features of `derive(Deserialize)` including
untagged enums.

You can opt back in to these impls in an unstable way by enabling the `"alloc"`
Cargo feature. This configuration provides integration for heap-allocated
collections without depending on the rest of the Rust standard library.

```toml
[dependencies]
serde = { version = "1.0", default-features = false, features = ["alloc"] }
```
