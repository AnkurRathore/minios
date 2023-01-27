# MINIOS - (Minimalist Kinux Kernel)
This project is the starting point of my Operating System desgin and implementation journey. A big thannks to Phill's [blog post](https://os.phil-opp.com).

This feature uses Rust Nightly. You can check out the [Rust Book]](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html#choo-choo-release-channels-and-riding-the-trains)

## Target Specification
The target specification is specified in a JSON file nameds as x86_64-blog_os.json and looks like:
`
{
    "llvm-target": "x86_64-unknown-none",
    "data-layout": "e-m:e-i64:64-f80:128-n8:16:32:64-S128",
    "arch": "x86_64",
    "target-endian": "little",
    "target-pointer-width": "64",
    "target-c-int-width": "32",
    "os": "none",
    "executables": true,
    "linker-flavor": "ld.lld",
    "linker": "rust-lld",
    "panic-strategy": "abort",
    "disable-redzone": true,
    "features": "-mmx,-sse,+soft-float"
}
`
## Building the Kernel
Bulding the kernel can be done using cargo build using the target specification file:

` cargo build --target x86_64-blog_os.json
`

## Running the Kernel

### Creating a Bootimage
To turn our compiled kernel into a bootable disk image, we need to link it with a bootloader.Instead of writing our own bootloader, which is a project on its own, we use the bootloader crate. This crate implements a basic BIOS bootloader without any C dependencies, just Rust and inline assembly. To use it for booting our kernel, we need to add a dependency on it:

`
# in Cargo.toml

[dependencies]
bootloader = "0.9.23"
`

Adding the bootloader as a dependency is not enough to actually create a bootable disk image. The problem is that we need to link our kernel with the bootloader after compilation, but cargo has no support for post-build scripts.

To solve this problem, we created a tool named bootimage that first compiles the kernel and bootloader, and then links them together to create a bootable disk image. To install the tool, execute the following command in your terminal:

`
cargo install bootimage
`
For running bootimage and building the bootloader, you need to have the llvm-tools-preview rustup component installed. You can do so by executing rustup component add llvm-tools-preview.

After installing bootimage and adding the llvm-tools-preview component, we can create a bootable disk image by executing:

`
cargo bootimage
`
The bootimage tool performs the following steps behind the scenes:

    1. It compiles our kernel to an ELF file.
    2. It compiles the bootloader dependency as a standalone executable.
    3. It links the bytes of the kernel ELF file to the bootloader.

When booted, the bootloader reads and parses the appended ELF file. It then maps the program segments to virtual addresses in the page tables, zeroes the .bss section, and sets up a stack. Finally, it reads the entry point address (our _start function) and jumps to it.

## Booting in QUEMU
We can now boot the disk image in a virtual machine. To boot it in [QEMU](https://www.qemu.org/download/#linux), execute the following command:

`
qemu-system-x86_64 -drive format=raw,file=target/x86_64-blog_os/debug/bootimage-minios.bin
`