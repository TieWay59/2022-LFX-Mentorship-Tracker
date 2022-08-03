# Week-09

## Progress

- Received feedback from the Rustls community.

## Discussion

Dirkjan Ochtman suggested that I refer to the integrated approach of the Quinn project.

The structure of the `crypto` module of the Quinn-proto is as follows.

```text
.
├── crypto
│   ├── ring.rs     # `use ring::{aead, hkdf, hmac};`
│   └── rustls.rs   # `use rustls::{self, ..}`
└── crypto.rs
```

In this project, they used conditional compilation to build a series of features.

```rust
/// Cryptography interface based on *ring*
#[cfg(feature = "ring")]
pub(crate) mod ring;
/// TLS interface based on rustls
#[cfg(feature = "rustls")]
pub mod rustls;
```

In this way, TLS is introduced as one of the options in the project's dependencies.

After my discussion with my mentor, I got a better understanding of the project design.

First, two kinds of features[^1] should be created, a ring and a Wasi-crypto.

```INI
# in cargo.toml

[dependencies]
ring = { .. , optional = true }
wasi_crypto = { .. , optional = true }

# this is a example for demonstration
[features]
default = ["ring"]
ring=["dep:ring"]
wasi-crypto=["dep:wasi_crypto"]
```

When compiling, you can choose `cargo build --features crypto` to compile the corresponding version.

In the project code, I will replace each ring call point with the corresponding programmatic implementation, e.g.

```rust
#[cfg(feature = "ring")]
use ring::aead;

#[cfg(feature = "wasi-crypto")]
use wasi_crypto::aead; // this is an imaginary structure
```

If I implement an interface similar to Ring, I can change the original logic of the code as little as possible.

But the WASI-crypto interface is still very low-level, so I may have to wrap a module inside the project:

```text
rustls
    ├── wasi_crypto_ringified
    │   ├── aead # use wasi_crypto
    │   └── ..
    └── ..
```

```rust
#[cfg(feature = "ring")]
use ring::aead;

#[cfg(feature = "wasi-crypto")]
use crate::wasi_crypto_ringified::aead; // this is an imaginary structure
```

This makes it easy for me to manage these implementations in one place.

My current preference is to use WASI-crypto to mimic the ring interface because of two things.

1. From what I know so far WASI-crypto is called in a more low-level way.
2. It minimizes the breaking change to the source code since a lot of files will need to be changed.

## Problems

There are many call points in the Rustls codebase that are redundant:

```rust
// in file `rustls/src/ticketer.rs`
use ring::aead; // <-- used here !

impl ProducesTickets for AeadTicketer {
    // --snip--

    fn encrypt(&self, message: &[u8]) -> Option<Vec<u8>> {
        // --snip--
        //           ↓ still write the complete path begin with ring.
        let nonce = ring::aead::Nonce::assume_unique_for_key(nonce_buf);
        let aad = ring::aead::Aad::empty();
        // --snip--
    }
```

In this case, I would change the absolute path to a relative path to call a method.
