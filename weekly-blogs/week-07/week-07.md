# Week-07

## Discussion

Mentor recommends running wasi-crypto first and running the test for sdk. While we think it's right to strive for code excellence, we don't want to be self-limiting, and making an imperfect first version is a big step forward.

If interfacing with the other community is temporarily impossible, our worst-case scenario is to use [cargo-patch](https://crates.io/crates/cargo-patch), which simply specifies that an updated codebase that has not been released to crate.io replaces the official version of a dependent library [^13].

## Problems

### `error: linking with cc failed: exit status: 1`

Simply put I broke the environment by installing it in an old container, which caused a complex `cc` compilation exception, and the conclusion was to reinstall the dev env.

I still don't know what caused it.

```log

warning: /root/wasi-crypto/implementations/bindings/rust/Cargo.toml: unused manifest key: build
warning: /root/wasi-crypto/implementations/bindings/rust/Cargo.toml: unused manifest key: target.wasm32-wasi.runner
   Compiling wasi-crypto-guest v0.1.1 (/root/wasi-crypto/implementations/bindings/rust)
error: linking with `cc` failed: exit status: 1
  |
  = note: "cc" "-m64" "/tmp/rustcLyq9zj/symbols.o"
  "/root/wasi-crypto/implementations/bindings/rust/target/debug/deps/wasi_crypto_guest-2134281d611eedb1.132ygmree055jths.rcgu.o"
  # --snip--
  "/root/wasi-crypto/implementations/bindings/rust/target/debug/deps/wasi_crypto_guest-2134281d611eedb1.22d67ggb01ik3shw.rcgu.o"
  "-Wl,--as-needed"
  "-L" "/root/wasi-crypto/implementations/bindings/rust/target/debug/deps"
  "-L" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-gnu/lib"
  "-Wl,-Bstatic" "/root/wasi-crypto/implementations/bindings/rust/target/debug/deps/libnum_enum-b7f74710f1c79e0b.rlib"
  "-Wl,--start-group"
  "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-gnu/lib/libstd-69edc9ac8de4d39c.rlib"
  # --snip--
  "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-gnu/lib/libcompiler_builtins-d1bd89f2a607e488.rlib"
  "-Wl,-Bdynamic" "-lgcc_s" "-lutil" "-lrt" "-lpthread" "-lm" "-ldl" "-lc" "-Wl,--eh-frame-hdr" "-Wl,-znoexecstack" "-L" "/root/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-gnu/lib" "-o" "/root/wasi-crypto/implementations/bindings/rust/target/debug/deps/wasi_crypto_guest-2134281d611eedb1" "-Wl,--gc-sections" "-pie" "-Wl,-zrelro,-znow" "-nodefaultlibs"
  = note: /usr/bin/ld: /root/wasi-crypto/implementations/bindings/rust/target/debug/deps/wasi_crypto_guest-2134281d611eedb1.48j7rig1gffj3rwj.rcgu.o: in function `wasi_crypto_guest::raw::options_open':
          /root/wasi-crypto/implementations/bindings/rust/src/raw.rs:746: undefined reference to `options_open'
          /usr/bin/ld: /root/wasi-crypto/implementations/bindings/rust/target/debug/deps/wasi_crypto_guest-2134281d611eedb1.48j7rig1gffj3rwj.rcgu.o: in function `wasi_crypto_guest::raw::options_close':
          /root/wasi-crypto/implementations/bindings/rust/src/raw.rs:761: undefined reference to `options_close'
          /usr/bin/ld: /root/wasi-crypto/implementations/bindings/rust/target/debug/deps/wasi_crypto_guest-2134281d611eedb1.48j7rig1gffj3rwj.rcgu.o: in function `wasi_crypto_guest::raw::options_set':
          /root/wasi-crypto/implementations/bindings/rust/src/raw.rs:781: undefined reference to `options_set'
          /usr/bin/ld: /root/wasi-crypto/implementations/bindings/rust/target/debug/deps/wasi_crypto_guest-2134281d611eedb1.48j7rig1gffj3rwj.rcgu.o: in function `wasi_crypto_guest::raw::symmetric_key_generate':
          /root/wasi-crypto/implementations/bindings/rust/src/raw.rs:2336: undefined reference to `symmetric_key_generate'
          /usr/bin/ld: /root/wasi-crypto/implementations/bindings/rust/target/debug/deps/wasi_crypto_guest-2134281d611eedb1.48j7rig1gffj3rwj.rcgu.o: in function `wasi_crypto_guest::raw::symmetric_key_close':
          /root/wasi-crypto/implementations/bindings/rust/src/raw.rs:2405: undefined reference to `symmetric_key_close'
          /usr/bin/ld: /root/wasi-crypto/implementations/bindings/rust/target/debug/deps/wasi_crypto_guest-2134281d611eedb1.48j7rig1gffj3rwj.rcgu.o: in function `wasi_crypto_guest::raw::symmetric_state_open':
          /root/wasi-crypto/implementations/bindings/rust/src/raw.rs:2776: undefined reference to `symmetric_state_open'
          /usr/bin/ld: /root/wasi-crypto/implementations/bindings/rust/target/debug/deps/wasi_crypto_guest-2134281d611eedb1.48j7rig1gffj3rwj.rcgu.o: in function `wasi_crypto_guest::raw::symmetric_state_close':
          /root/wasi-crypto/implementations/bindings/rust/src/raw.rs:2868: undefined reference to `symmetric_state_close'
          /usr/bin/ld: /root/wasi-crypto/implementations/bindings/rust/target/debug/deps/wasi_crypto_guest-2134281d611eedb1.48j7rig1gffj3rwj.rcgu.o: in function `wasi_crypto_guest::raw::symmetric_state_max_tag_len':
          /root/wasi-crypto/implementations/bindings/rust/src/raw.rs:3008: undefined reference to `symmetric_state_max_tag_len'
          /usr/bin/ld: /root/wasi-crypto/implementations/bindings/rust/target/debug/deps/wasi_crypto_guest-2134281d611eedb1.48j7rig1gffj3rwj.rcgu.o: in function `wasi_crypto_guest::raw::symmetric_state_encrypt':
          /root/wasi-crypto/implementations/bindings/rust/src/raw.rs:3043: undefined reference to `symmetric_state_encrypt'
          /usr/bin/ld: /root/wasi-crypto/implementations/bindings/rust/target/debug/deps/wasi_crypto_guest-2134281d611eedb1.48j7rig1gffj3rwj.rcgu.o: in function `wasi_crypto_guest::raw::symmetric_state_decrypt':
          /root/wasi-crypto/implementations/bindings/rust/src/raw.rs:3121: undefined reference to `symmetric_state_decrypt'
          collect2: error: ld returned 1 exit status

  = help: some `extern` functions couldn't be found; some native libraries may need to be installed or have their path specified
  = note: use the `-l` flag to specify native libraries to link
  = note: use the `cargo:rustc-link-lib` directive to specify the native libraries to link with Cargo (see https://doc.rust-lang.org/cargo/reference/build-scripts.html#cargorustc-link-libkindname)

error: could not compile `wasi-crypto-guest` due to previous error
```

I searched the following libraries via `ldconfig -p | grep xxx` and they all exist.

```text
  "-lgcc_s"
  "-lutil"
  "-lrt"
  "-lpthread"
  "-lm"
  "-ldl"
  "-lc"
```

I reached out to my mentor and it seems a lot easier to reinstall everyting.

### Special error `E: You don't have enough free space in /var/cache/apt/archives/.`

One possibility is that docker takes up too much space, but I didn't dare to `docker system prune`, so tried deleting a large image but it didn't work. I have rebooted and found `df -h` overlay reduced by 10G, definitely a lot smaller.

Conclusion: estimated that the container is broken, pull & run again.

[^13]: [Overriding Dependencies - The Cargo Book (rust-lang.org)](https://doc.rust-lang.org/cargo/reference/overriding-dependencies.html#working-with-an-unpublished-minor-version)
