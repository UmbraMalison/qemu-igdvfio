qemu-igdvfio
============

Extending VFIO framework to support Intel Integrated Graphics Devices


The code in this project is specifically written for the following platform:
* INTEL DQ67OW
*   HOST-BRIDGE: 8086:0100
*   GPU: 8086:0102
*   LPC: 8086:1c4e
*   SMBUS: 8086:1c22


Example qemu-command-line:

qemu-system-x86_64 -cpu host -M q35 -L biosdir -bios biosdir/bios.bin -acpitable file=biosdir/q35-acpi-dsdt.aml -m 2048 -enable-kvm -device vfio-pci,host=00:02.0,id=vga1,x-vga=on,addr=2.0 -usb -drive file=../GUEST.qcow2,if=virtio,snapshot=on -vga none

Host Kernel-command-line should contain:
* intel_iommu=on,igfx_off
* modprobe.blacklist=i915,intel_gtt,intel_agp,drm


STATUS
======
Intelfb appears to be working. Software rendering appears to be working.

Hardware accleration is not working.

Errors are, STUCK on BLITTER RING and FAILED to RESET device.

    
Build Instructions
==================

qemu
----

$ git clone git://git.qemu.org/qemu.git

$ cd qemu

$ git apply qemu_igd.patch

$ ./configure <add custom options here>

e.g. (archlinux)
$ ./configure --prefix=/usr --sysconfdir=/etc --python=/usr/bin/python2 --smbd=/usr/bin/smbd --enable-docs --libexecdir=/usr/lib/qemu --audio-drv-list=alsa,sdl,pa --enable-gtk --with-gtkabi=3.0 --enable-linux-aio --enable-seccomp --enable-spice --disable-werror --enable-debug
              
$ make

seabios
-------

$ git clone git://git.qemu.org/seabios.git

$ cd ..

$ cd seabios

$ git apply seabios_igd.patch

$ make

$ mkdir ../biosdir

$ cp ../qemu/pc-bios/*.bin ../biosdir

$ cp out/bios.bin ../biosdir

$ cp out/src/fw/*.aml ../biosdir

done
