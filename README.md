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

### `Embed.toml`
This file is used to configure `cargo embed`. It is added, with:
- `chip = "STM32F303VCTx"`: specify the chip type.
- `[default.reset]`: whether to halt after flashing. This can be useful for debugging the startup process.

### `.cargo/config.toml`
This file describes the linker to use. In this case, the `rust-lld` linker is used.

## Working with the executable

### Building
The build the binary, the command `cargo build --target thumbv7em-none-eabi` is used. This places the binary in a location like `target/thumbv7em-none-eabi/debug/rust` (running `cargo run` instead of `cargo build` will fail, but will show the path to the executable).

The executable can be checked with `readelf -h target/thumbv7em-none-eabi/debug/rust`.

### Flashing
`probe-rs list` can be used to list the `probe`-able devices (`probe-rs info` for more information).

If bad code is present on the chip, it may prevent `cargo embed` from flashing the new binary. In this case, use `openocd -f interface/stlink.cfg -f target/stm32f3x.cfg` to gain access to the chip, followed by `telnet localhost 4444` (in a different shell). This opens a new prompt, in which the chip can be reset:
```
> reset halt
> stm32f1x unlock 0
> reset halt
> exit
```
In the prompt, individual values at specific memory locations can be read using:
```
> mdw <address> <nb of words>  # e.g. mdw 0x08000000 4
0x08000000: 20010000 08000195 08001f4d 08003a3f
```

To flash the binary to the chip:
1. Use `openocd -f interface/stlink.cfg -f target/stm32f3x.cfg` to establish the connection with the microcontroller.
2. Launch the GDB server with `gdb <path to binary>` (in a different shell).
3. Flash the binary:
   ```
   > target extended-remote :3333
   > load
   > monitor reset halt
   > continue
   ```

### Debugging
The `openocd` command above will also open a GDB server. This can be accessed with `gdb <binary path>`, which will open a prompt. Here are some useful commands to use in this prompt:
- `target extended-remote :3333`: connects to the GDB server.
- `break main`: sets a breakpoint at the main function (highly recommended).
- `layout src`: open a TUI to view the code.
- `tui disable`: close the code TUI.
- `break <linenb>`: set a breakpoint at the specified line.
- `print <variable>`: prints the current value of a variable.
- `info locals`: prints information about the local variables.
- `next`: move to the next line of code.
- `info functions`: shows the function symbols seen by GDB.

