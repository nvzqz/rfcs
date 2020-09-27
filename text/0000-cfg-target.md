- Feature Name: `cfg-target`
- Start Date: 2020-09-27
- RFC PR: [rust-lang/rfcs#0000](https://github.com/rust-lang/rfcs/pull/0000)
- Rust Issue: [rust-lang/rust#0000](https://github.com/rust-lang/rust/issues/0000)

# Summary
[summary]: #summary

This proposes a new `cfg`: `target`, which matches the entire target triple
string (e.g. `arm-unknown-linux-gnueabihf`). This also adds a `CARGO_CFG_TARGET`
environment variable for parity with other `CARGO_CFG_*` variables.

# Motivation
[motivation]: #motivation

To `#[cfg]` against a specific target, a `build.rs` script is required to emit a
custom `cfg` based on the `TARGET` environment variable. Adding a build script
increases compile time and makes a crate incompatible with certain build
systems.

Otherwise, all available components would need to be specified separately:
`target_arch`, `target_vendor`, `target_os`, and `target_env`. This can be very
cumbersome. Note that the target ABI cannot currently be `#[cfg]`-ed against, so
a `build.rs` is still necessary to match all target components.

# Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

This would act like existing `target_*` configurations but match against all
components (except `target_feature`).

```rust
#[cfg(target = "x86_64-apple-ios-macabi")]
mod mac_catalyst;
```

This includes `#[cfg_attr(target = "...", attr)]`.

# Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

`target` is a key-value option set once with the target's Rust triple.

Example values:

- `"aarch64-apple-darwin"`
- `"arm-unknown-linux-gnueabihf"`
- `"x86_64-apple-ios-macabi"`
- `"x86_64-pc-windows-gnu"`
- `"x86_64-pc-windows-msvc"`
- `"x86_64-unknown-linux-gnu"`

# Drawbacks
[drawbacks]: #drawbacks

- Configuring against specific targets can be overly strict and could make
  certain `#[cfg]`s miss similar configurations with small changes.

  For example: `aarch64-unknown-none` does not match
  `aarch64-unknown-none-softfloat`, yet one would likely want to include ABI
  variants. The same concern applies to the target vendor.

  A potential solution would be to allow glob matching (e.g.
  `aarch64-unknown-none*`), but that is not within the scope of this proposal
  because it is not currently used in other `#[cfg]`s.

- The `CARGO_CFG_TARGET` environment variable is redundant with the existing
  `TARGET`. However, including it would be consistent with other `CARGO_CFG_*`
  variables.

# Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

We can keep the existing work-around of checking the `TARGET` environment
variable in a `build.rs` script. However, that increases compile time and makes
a crate incompatible with certain build systems.

# Prior art
[prior-art]: #prior-art

- [Target component configurations](https://doc.rust-lang.org/reference/conditional-compilation.html#set-configuration-options):
  `target_arch`, `target_vendor`, `target_os`, and `target_env`.

- `TARGET` and `CARGO_CFG_TARGET_*`
  [environment variables for `build.rs`](https://doc.rust-lang.org/cargo/reference/environment-variables.html#environment-variables-cargo-sets-for-build-scripts).

# Unresolved questions
[unresolved-questions]: #unresolved-questions

- How do we ensure a project does not miss configurations similar to the ones
  being `#[cfg]`-ed against with this feature? Perhaps this should be added as a
  Clippy lint that's off by default.

# Future possibilities
[future-possibilities]: #future-possibilities

This would enable `#[cfg]`-ing against a specific target ABI (e.g. `macabi`,
`eabihf`). However, that is not the motivation for this proposal and should be
handled separately.
