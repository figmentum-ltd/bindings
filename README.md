# Rust Bindings for creating Platform compliant WASM components #
This repository is where important code for communication with Wasm components is being extracted. It contains already generated host and guest Rust bindings. The bindings are based on the Wasm component's IDL - [WIT](https://github.com/figmentum-ltd/bindings/blob/main/wit/world.wit).

## About ##
The end-goal of `bindings` is to facilitate the creation of components, which can be used as pat of Figmentum's `Platform` project. Instead of each component generating bindings and having to resolve errors, `bindings` provides a hassle-free solution with the latest version of the .wit file guest bindings.\
Once a component is created using these bindings, it can be called from the `Platform` and use the provided resources. The entry points of each component are labeled as `export` and outside resources can be accessed through functions/interfaces labeled as `import` (see [world.wit](https://github.com/figmentum-ltd/bindings/blob/main/wit/world.wit)). 

## Creating a Component: the easy way ##
> Prerequisites: You need to have the `rust-analyzer` extension and optionally `cargo component` binary installed for easier component creation and build. Otherwise, other tools can be used such as `wasm-tools`.

1. Use `$ cargo component new --lib <component_name>` to create a new library component.
2. Since the bindings will be used externally, delete the `/wit` directory.
3. Add this repository as a dependency in your `Cargo.toml` file under `[dependencies]`.
    ```toml
    [dependencies]
    bindings = { version = "0.x", git = "https://github.com/figmentum-ltd/bindings.git", features = ["guest"] }
    ```
    > Note: Remove other dependencies added by `cargo component new` command such as `wasm-bindgen-rt`.

    > Note: The version of `bindings` should be the latest major version available in this repository to comply with `Platform`'s requirements for compatibility.

4. You will use the just added `bindings` dependency by following the steps below:
- Create a struct in your `lib.rs` file that will represent the component (e.g. `Component`)
```Rust
// lib.rs
use bindings::guest::{Guest, ByteArray, Event};

struct Component;
```
- Implement the `Guest` trait for the struct. The `init` function is called when the component is created. if using `rust-analyzer` to fill the functions. Remove the module `_rt::` from types such as Result, Vec, etc.
```Rust
impl Guest for Component {
    fn execute(cmd: ByteArray) -> Result<Vec<Event>, Vec<u8>> {
        todo!()
    }

    fn query(req: ByteArray) -> Result<ByteArray, ByteArray> {
        todo!()
    }
}
```
- Required macro for building the component. The struct which implements the `Guest` trait should be passed to the `export!` macro.
```Rust
bindings::export!(Component with_types_in bindings::guest);
```
5.  Build the component using `$ cargo component build --target wasm32-unknown-unknown`.
> Note: If you don't have the target added, you can resolve this by running `$ rustup target add wasm32-unknown-unknown`.
6.  The generated `.wasm` file can be found in ./target/wasm32-unknown-unknown/release/<component_name>.wasm.
7. **Congratulations!** You have successfully created a component that can be used by the `Platform`.

### Notes ###
- Components by `Platform` design don't have direct access to `std` library. Instead, some handy `std` features can be accessed through the component's `import interface`. 
Thus the component is built targeting `wasm32-unknown-unknown` for simplicity and performance.

   