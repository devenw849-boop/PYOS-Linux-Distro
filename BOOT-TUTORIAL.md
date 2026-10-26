Short doc block you can paste
PyOS is a prototype x86 operating system built around a custom kernel, FAT16 filesystem support, GRUB booting, and a Python userspace stack. It is designed to behave like a small distro foundation, but it is not yet a full production-ready Linux distribution.

Boot methods:

make run
make run-graphics
make iso
qemu-system-i386 -cdrom pyos.iso -boot d
qemu-system-i386 -cdrom pyos.iso -drive file=build/pyos-disk.img,format=raw,if=ide,index=0,media=disk -boot d
NOTE: when none of the boot options work, blame the code i wrote, qemu virt manager and vbox were both not mounting FAT16 for some weird f***ing reason
