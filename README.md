# Custom OS

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
Main article: Calling Global Constructors
This tutorial showed a small example of how to create a minimal environment for C and C++ kernels. Unfortunately, you don't have everything set up yet. For instance, C++ with global objects will not have their constructors called because you never do it. The compiler uses a special mechanism for performing tasks at program initialization time through the crt*.o objects, which may be valuable even for C programmers. If you combine the crt*.o files correctly, you will create an _init function that invokes all the program initialization tasks. Your boot.o object file can then invoke _init before calling kernel_main.

## Links

[wiki OS Dev](https://wiki.osdev.org/Bare_Bones)
