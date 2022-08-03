# Week-08

## Progress

- Setting up wasi-crypto dev environment.

## dev log

> The following workspace is in my personal wasmedge container.

1. Compile the toolchain for WasmEdge's proposal/wasi-crypto branch [^2] wasi_crypto. (Note that the plugin's compilation parameters can be found in the project's CMakeLists [^3], which defaults to the OFF state [^6], and in the build tutorial [^5] [^7])

   1. Check `/WasmEdge/CMakeLists.txt` for the corresponding plugin `option(WASMEDGE_PLUGIN_WASI_CRYPTO "Enable WasmEdge Wasi-crypto plugin." OFF)` is off, you have to turn it on manually when running the command .

   2. ```bash
      mkdir -p tw59build && cd tw59build
      cmake -DCMAKE_BUILD_TYPE=Release -DWASMEDGE_BUILD_AOT_RUNTIME=OFF -DWASMEDGE_BUILD_TESTS=OFF -DWASMEDGE_PLUGIN_WASI_CRYPTO=ON  .. && make -j8
      # Where `-DWASMEDGE_PLUGIN_WASI_CRYPTO=ON` is required, corresponding to the first step ref [^7].
      # where `-DWASMEDGE_BUILD_AOT_RUNTIME=OFF -DWASMEDGE_BUILD_TESTS=OFF` can be built faster, refer to [^5]
      ```

   3. In the current build directory `cmake --install .` [^7]

   4. Verify that the build was successful and that the path `tw59build/tools/wasmedge` exists.

2. Clone the main branch of the official wasi-crypto [^1] (leave the other branch alone for now) and try `cargo test`

   1. Run `rustup target add wasm32-wasi`

   2. Edit `implementations/bindings/rust/Cargo.toml`

      ```toml
      # Append the following two subsections
      [build]
      target="wasm32-wasi"

      [target.wasm32-wasi]
      runner = "~/WasmEdge/tw59build/tools/wasmedge/wasmedge"
      ```

   3. Run `cargo test`

   4. ```log
      error: failed to parse manifest at `/root/wasi-crypto/implementations/bindings/rust/Cargo.toml`

      Caused by:
      the cargo feature `per-package-target` requires a nightly version of Cargo, but this is the `stable` channel
      See https://doc.rust-lang.org/book/appendix-07-nightly-rust.html for more information about Rust release channels.
      See https://doc.rust-lang.org/cargo/reference/unstable.html#per-package-target for more information about using this feature.
      ```

      Here you need to change rust nightly so `rustup override set nightly` (of course you can change back to `stable`) [^9]

      It will compile, but rust-analyzer needs to change rustup default `rustup default nightly` [^10]

   5. ```log
      warning: /root/wasi-crypto/implementations/bindings/rust/Cargo.toml: unused manifest key: build
      warning: /root/wasi-crypto/implementations/bindings/rust/Cargo.toml: unused manifest key: target.wasm32-wasi.runner
         Finished test [unoptimized + debuginfo] target(s) in 0.03s
         Running unittests src/lib.rs (/root/wasi-crypto/target/wasm32-wasi/debug/deps/wasi_crypto_guest-5e1a078bb78b42fe.wasm)
      /root/wasi-crypto/target/wasm32-wasi/debug/deps/wasi_crypto_guest-5e1a078bb78b42fe.wasm: 1: Syntax error: end of file unexpected
      error: test failed, to rerun pass '--lib'
      ```

      So far such a return is as expected. The analysis shows that the test is ending early, but because it is not perfect, it does not show back the specific error to the terminal.

      The results can be seen by manually running the product `wasmedge /root/wasi-crypto/target/wasm32-wasi/debug/deps/wasi_crypto_guest-5e1a078bb78b42fe.wasm`.

      ```log
      running 7 tests
      test test::test::test_rsa_signatures ... ok
      test test::test::test_signatures ... ok
      test test::test::test_symmetric ... Error: UnsupportedAlgorithm
      [2022-07-27 19:24:05.981] [error] execution failed: unreachable, Code: 0x89
      [2022-07-27 19:24:05.981] [error]     In instruction: unreachable (0x00) , Bytecode offset: 0x00059719
      [2022-07-27 19:24:05.981] [error]     When executing function name: "_start"
      ```

      Simply speaking, there exist some algorithms that are not yet supported and can comment out the corresponding tests.

[^1]: [wasi-crypto/implementations/hostcalls/rust at main - WebAssembly/wasi-crypto (github.com)](https://github.com/WebAssembly/wasi-crypto/tree/main/implementations/hostcalls/rust)
[^2]: [WasmEdge/plugins/wasi_crypto at proposal/wasi-crypto - WasmEdge/WasmEdge (github.com)](https://github.com/WasmEdge/WasmEdge/tree/proposal/wasi-crypto/plugins/wasi_crypto)
[^3]: [WasmEdge/CMakeLists.txt at master - WasmEdge/WasmEdge (github.com)](https://github.com/WasmEdge/WasmEdge/blob/master/CMakeLists.txt)
[^5]: [Build WasmEdge from source - WasmEdge Runtime](https://wasmedge.org/book/en/extend/build.html)
[^6]: [WasmEdge/CMakeLists.txt at master - WasmEdge/WasmEdge (github.com)](https://github.com/WasmEdge/WasmEdge/blob/f06e03b9617301aba1c9d3b8f9ca86b8d6d8beb0/CMakeLists.txt#L72)
[^7]: [Crypto for WASI - WasmEdge Runtime](https://wasmedge.org/book/en/dev/rust/wasicrypto.html#wasmedge-building-and-installation)
[^9]: [How to switch between Rust toolchains?](https://stackoverflow.com/a/65644807/15213543)
[^10]: [Rust Analyzer with unstable features?](https://www.reddit.com/r/rust/comments/fx4ikg/comment/fms4omv/?utm_source=share&utm_medium=web2x&context=3)
