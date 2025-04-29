# STM32F3 Discovery with Rust

## Microcontroller
The STM32F3 Discovery board uses the STM32F303VCT6 chip, with the core based on ARM Cortex-M4.

## Changes to base nix starter
Starting from the basic [Rust nix starter](https://github.com/jacg/nix-starters), the following changes are made in order for embedded work on the STM32F3 Discovery board to be possible.

### `flake.nix`
The only changes made here were the addition of `cargo-binutils` (for LLVM tools), `probe-rs-tools` (embedded toolkit, for on-chip debugging and flashing), and `gdb` (for debugging).

### `rust-toolchain.toml`
The compilation target architecture for the Cortex-M4 is "thumbv7em-none-eabi". This is added to the `targets` parameter in `rust-toolchain.toml`.
