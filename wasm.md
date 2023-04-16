# WebAssembly / WASM

## Using with rust

- Install [wasm-pack](https://rustwasm.github.io/wasm-pack/installer/) for building, testing and publishing Rust-generated WebAssembly.
- Install `cargo-generate` via `cargo install cargo-generate` for using existing Rust projects as templates for new projects
- `npm`

## Runtimes

WebAssembly also has the potential to run very lightweighted outside the browser. Therefore, an interface called `wasi` is under creation. It enables the development of runtimes outside the browser. Currently, there are two projects:

- [wasmtime](https://github.com/bytecodealliance/wasmtime) is the project by the [bytecodealliance](https://bytecodealliance.org/) which is behind the wasm specs
- [wasmer](https://github.com/wasmerio/wasmer) under the MIT License supports more languages and claims to be faster
