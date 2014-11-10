qemu-igdvfio
============


This Project has moved. The information here may not be correct and the code is not maintained.

The new project can be found here:
www.outsideglobe.com/igdvfio





Extending VFIO framework to support Intel Integrated Graphics Devices


The code in this project is specifically written for the following platform:
* INTEL DQ67OW
*   HOST-BRIDGE: 8086:0100
*   GPU: 8086:0102
*   LPC: 8086:1c4e

It should however work with any Core i IGD platform, providing the SeaBIOS hack (described below) is done correctly.

Kernel
------

Please note!!
Since kernel 3.15, kernel command line option igfx_off causes boot up problems.
please use latest 3.14


Usage
=====

Example qemu-command-line:

    qemu-system-x86_64 -cpu host -M q35 -L biosdir -bios biosdir/bios.bin -acpitable file=biosdir/q35-acpi-dsdt.aml -m 2048 -enable-kvm -device vfio-pci,host=00:02.0,id=vga1,x-vga=on,addr=2.0,romfile=biosdir/IGDQ67.rom -usb -drive file=../GUEST.qcow2,if=virtio,snapshot=on -vga none

Host cmdline
------------

Host Kernel-command-line should contain:

    intel_iommu=on,igfx_off, modprobe.blacklist=i915,intel_gtt,intel_agp,drm

SeaBIOS hack
------------

For an unmodified guest, changes need to be made in seabios and qemu to accept at least the LPC device ID of your host.

seabios/src/fw/dev-q35.h

    #define PCI_DEVICE_ID_INTEL_ICH9_LPC      0x(HOST LPC)

Bios
=========

[How to dump the video bios](https://01.org/linuxgraphics/documentation/how-dump-video-bios-0 "Video Bios")

Alternative

    dd if=/dev/mem of=vbios.dump bs=64k skip=12 count=1


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
    $ git apply qemu-stable-2.1-igdvfio.patch
    $ ./configure --disable-werror
    $ make
    $ cd ..
    
e.g. (archlinux)

    $ ./configure --prefix=/usr --sysconfdir=/etc --python=/usr/bin/python2 --smbd=/usr/bin/smbd --enable-docs --libexecdir=/usr/lib/qemu --audio-drv-list=alsa,sdl,pa --enable-gtk --with-gtkabi=3.0 --enable-linux-aio --enable-seccomp --enable-spice --disable-werror --enable-debug

SeaBIOS
-------

    $ git clone git://git.qemu.org/seabios.git
    $ cd seabios
    $ git apply seabios-1.7.5-stable-igdvfio.patch
    $ make
    $ mkdir ../biosdir
    $ cp ../qemu/pc-bios/*.bin ../biosdir
    $ cp out/bios.bin ../biosdir
    $ cp out/src/fw/*.aml ../biosdir

