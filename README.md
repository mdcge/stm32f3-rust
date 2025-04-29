# STM32F3 Discovery with Rust

## Microcontroller
The STM32F3 Discovery board uses the STM32F303VCT6 chip, with the core based on ARM Cortex-M4.

## Changes to base nix starter
Starting from the basic [Rust nix starter](https://github.com/jacg/nix-starters), the following changes are made in order for embedded work on the STM32F3 Discovery board to be possible.

### `flake.nix`
The only changes made here were the addition of
- `cargo-binutils`: for LLVM tools.
- `probe-rs-tools`: embedded toolkit, for on-chip debugging and flashing.
- `gdb`: for debugging.

### `rust-toolchain.toml`
The compilation target architecture for the Cortex-M4 is "thumbv7em-none-eabi". This is added to the `targets` parameter in `rust-toolchain.toml`.

### `Cargo.toml`
The `Cargo.toml` file was update to include these crates:
- `cortex-m-rt`: provides minimal startup code and runtime environment.
- `cortex-m`: provides access to common Cortex-M core functionalities.
- `panic-halt`: sets the panicking behaviour to halt.
- `stm32fxx-hal`, with `rt` and `stm32f303xc` enabled: the HAL for the STM32F family, specifically the STM32F303 version. The `rt` refers to the startup functionalities.

## Building the executable
The build the binary, the command `cargo build --target thumbv7em-none-eabi` is used. This places the binary in a location like `target/thumbv7em-none-eabi/debug/rust` (running `cargo run` instead of `cargo build` will fail, but will show the path to the executable).

The executable can be checked with `readelf -h target/thumbv7em-none-eabi/debug/rust`.
