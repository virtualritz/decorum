![Decorum](https://raw.githubusercontent.com/olson-sean-k/decorum/master/doc/decorum.png)

**Decorum** is a Rust library that provides ordering, equality, and hashing for
floating point types.

[![Build Status](https://travis-ci.org/olson-sean-k/decorum.svg?branch=master)](https://travis-ci.org/olson-sean-k/decorum)
[![Documentation](https://docs.rs/decorum/badge.svg)](https://docs.rs/decorum)
[![Crate](https://img.shields.io/crates/v/decorum.svg)](https://crates.io/crates/decorum)

## Constrained Wrapper Types

The `NotNan` and `Finite` types wrap raw floating point values and disallow
certain values like `NaN`, `INF`, and `-INF`. These type implement all of the
standard operation traits like `Add` and `Mul`, as well as numeric traits from
the [num-traits](https://crates.io/crate/num-traits) crate. More targeted
traits that complement the `Float` trait are also introduced: `Nan`,
`Infinite`, and `Real`.

Wrapper types also implement `Eq`, `Hash`, and `Ord`. Hashing uses a
canonicalized form that normalizes `NaN` values and expands all floating point
values into a 64-bit sequence. A similar approach is used for `Eq`, normalizing
`NaN` values. Because `NotNan` and `Finite` disallow `NaN` values, they support
`Ord`.

## Disabling Constraints

Constraint checking can be toggled with the `enforce-constraints` feature. This
is useful if code would like to enforce constraints for some builds but not
others (e.g., debug vs. release builds). Constraint checking is enabled by
default.

This predominantly affects the behavior of the `from_raw_float` function, which
will immediately panic for disallowed values when the feature is enabled, but
will perform no checks when the feature is disabled (which can lead to latent
panics and unexpected behavior).

This feature should be enabled unless performance is a concern.

## Hashing Functions

The `NotNan` and `Finite` types implement `Hash`, but sometimes it is not
possible or ergonomic to use these. Hashing functions for raw floating point
values can be used instead.

With the [derivative](https://crates.io/crates/derivative) crate, floating
point fields can be hashed using one of these functions when deriving `Hash`.
For example, a `Vertex` type used by a rendering pipeline could use this for
floating point fields:

```rust
use decorum;

#[derive(Derivative)]
#[derivative(Hash)]
pub struct Vertex {
    #[derivative(Hash(hash_with = "decorum::hash_float_array"))]
    pub position: [f32; 3],
    ...
}
```

Scalar values, slices, and arrays up to length 16 are supported.
