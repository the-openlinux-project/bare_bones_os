# Custom OS

## Setting up environment

### Preparation

Download Binutils and GCC of any version(Latest version is recommanded).

Export The following,

`export PREFIX="$HOME/opt/cross"`

`export TARGET=i686-elf`

`export PATH="$PREFIX/bin:$PATH"`

### Binutils

Build binutils

`cd $HOME/src`

`mkdir build-binutils`

`cd build-binutils`

`../binutils-x.y.z/configure --target=$TARGET --prefix="$PREFIX" --with-sysroot --disable-nls --disable-werror`

`make`

`make install`

This compiles the binutils (assembler, disassembler, and various other useful stuff), runnable on your system but handling code in the format specified by $TARGET.

--disable-nls tells binutils not to include native language support. This is basically optional, but reduces dependencies and compile time. It will also result in English-language diagnostics, which the people on the Forum understand when you ask your questions. ;-)

--with-sysroot tells binutils to enable sysroot support in the cross-compiler by pointing it to a default empty directory. By default, the linker refuses to use sysroots for no good technical reason, while gcc is able to handle both cases at runtime. This will be useful later on.

### GCC

Build GCC

`cd $HOME/src`

##### The $PREFIX/bin dir _must_ be in the PATH. We did that above.
`which -- $TARGET-as || echo $TARGET-as is not in the PATH`

`mkdir build-gcc`

`cd build-gcc`

`../gcc-x.y.z/configure --target=$TARGET --prefix="$PREFIX" --disable-nls --enable-languages=c,c++ --without-headers`

`make all-gcc`

`make all-target-libgcc`

`make install-gcc`

`make install-target-libgcc`

We build libgcc, a low-level support library that the compiler expects available at compile time. Linking against libgcc provides integer, floating point, decimal, stack unwinding (useful for exception handling) and other support functions. Note how we are not simply running make && make install as that would build way too much, not all components of gcc are ready to target your unfinished operating system.

--disable-nls is the same as for binutils above.

--without-headers tells GCC not to rely on any C library (standard or runtime) being present for the target.

--enable-languages tells GCC not to compile all the other language frontends it supports, but only C (and optionally C++).

It will take a while to build your cross-compiler.

If you are building a cross compiler for x86-64, you may want to consider building Libgcc without the "red zone": Libgcc_without_red_zone

### Using the new Compiler

`$HOME/opt/cross/bin/$TARGET-gcc --version`

`export PATH="$HOME/opt/cross/bin:$PATH"`

## Compilation

At First, compile boot.s

`i686-elf-as boot.s -o boot.o`

then, the kernel (kernel.c)

`i686-elf-gcc -c kernel.c -o kernel.o -std=gnu99 -ffreestanding -O2 -Wall -Wextra`

if kernel is written i cpp,

`i686-elf-g++ -c kernel.c++ -o kernel.o -ffreestanding -O2 -Wall -Wextra -fno-exceptions -fno-rtti`

then, Linking boot.s and kernel.c with linker.ld

`i686-elf-gcc -T linker.ld -o myos.bin -ffreestanding -O2 -nostdlib boot.o kernel.o -lgcc`

## Verifying Multiboot

`grub-file --is-x86-multiboot myos.bin`

or

`if grub-file --is-x86-multiboot myos.bin; then
  echo multiboot confirmed
else
  echo the file is not multiboot
fi`

## Booting the Kernel

### create a file named 'grub.cfg' with content,

``menuentry "myos" {
	multiboot /boot/myos.bin
}``

Then,

`mkdir -p <isodir>/boot/grub`

`cp myos.bin <isodir>/boot/myos.bin`

`cp grub.cfg <isodir>/boot/grub/grub.cfg`

`grub-mkrescue -o myos.iso <isodir>`


## Testing your operating system (QEMU)

Runnning the ISO,

`qemu-system-i386 -cdrom myos.iso`

Runnning the Kernel,

`qemu-system-i386 -kernel myos.bin`

## Testing your operating system (Real Hardware)

Runnning on Real Hardware,

`sudo dd if=myos.iso of=/dev/sdx && sync`

## Moving Forward

Now that you can run your new shiny operating system, congratulations! Of course, depending on how much this interests you, it may just be the beginning. Here's a few things to get going.

### Adding Support for Newlines to Terminal Driver
The current terminal driver does not handle newlines. The VGA text mode font stores another character at the location, since newlines are never meant to be actually rendered: they are logical entities. Rather, in terminal_putchar check if c == '\n' and increment terminal_row and reset terminal_column.

### Implementing Terminal Scrolling
In case the terminal is filled up, it will just go back to the top of the screen. This is unacceptable for normal use. Instead, it should move all rows up one row and discard the upper most, and leave a blank row at the bottom ready to be filled up with characters. Implement this.

### Rendering Colorful ASCII Art
Use the existing terminal driver to render some pretty stuff in all the glorious 16 colors you have available. Note that only 8 colors may be available for the background color, as the uppermost bit in the entries by default means something other than background color. You'll need a real VGA driver to fix this.

### Calling Global Constructors
Main article: [Calling Global Constructors](https://wiki.osdev.org/Calling_Global_Constructors)
This tutorial showed a small example of how to create a minimal environment for C and C++ kernels. Unfortunately, you don't have everything set up yet. For instance, C++ with global objects will not have their constructors called because you never do it. The compiler uses a special mechanism for performing tasks at program initialization time through the crt*.o objects, which may be valuable even for C programmers. If you combine the crt*.o files correctly, you will create an _init function that invokes all the program initialization tasks. Your boot.o object file can then invoke _init before calling kernel_main.

## Links

[wiki OS Dev](https://wiki.osdev.org/Bare_Bones)
