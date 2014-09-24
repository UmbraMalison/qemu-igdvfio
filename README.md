qemu-igdvfio
============

Extending VFIO framework to support Intel Integrated Graphics Devices


The code in this project is specifically written for the following platform:
* INTEL DQ67OW
*   HOST-BRIDGE: 8086:0100
*   GPU: 8086:0102
*   LPC: 8086:1c4e


Usage
=====

Example qemu-command-line:

    qemu-system-x86_64 -cpu host -M q35 -L biosdir -bios biosdir/bios.bin -acpitable file=biosdir/q35-acpi-dsdt.aml -m 2048 -enable-kvm -device vfio-pci,host=00:02.0,id=vga1,x-vga=on,addr=2.0,romfile=biosdir/IGDQ67.rom -usb -drive file=../GUEST.qcow2,if=virtio,snapshot=on -vga none

Host cmdline
------------

Host Kernel-command-line should contain:

    intel_iommu=on,igfx_off, modprobe.blacklist=i915,intel_gtt,intel_agp,drm

Un-Modified Guest
-----------------

For an unmodified guest, changes need to be made in seabios and qemu to accept at least the LPC device ID of your host.

qemu/include/hw/pci/pci_ids.h

    #define PCI_DEVICE_ID_INTEL_ICH9_6    0x(HOST SMBUS)
    #define PCI_DEVICE_ID_INTEL_ICH9_8    0x(HOST LPC)
    #define PCI_DEVICE_ID_INTEL_Q35_MCH   0x(HOST HOST-BRIDGE)

qemu/hw/isa/lpc_ich9.c

    k->vendor_id = host_pci_read_config(0,0x1f,0,0x00,2);
    k->device_id = host_pci_read_config(0,0x1f,0,0x02,2);
    k->revision = host_pci_read_config(0,0x1f,0,0x08,1);

qemu/hw/pci-host/q35.c

    k->vendor_id = host_pci_read_config(0,0,0,0x00,2);
    k->device_id = host_pci_read_config(0,0,0,0x02,2);
    k->device_id = host_pci_read_config(0,0,0,0x02,2);

seabios/src/fw/dev-q35.h

    #define PCI_DEVICE_ID_INTEL_ICH9_SMBUS    0x(HOST SMBUS)
    #define PCI_DEVICE_ID_INTEL_ICH9_LPC      0x(HOST LPC)
    #define PCI_DEVICE_ID_INTEL_Q35_MCH       0x(HOST HOST-BRIDGE)


Modified Guest
--------------

For a modified guest, which may be more successful as QEMU will be running as it should, you will need to modify the guest kernels i915 driver to accept QEMU's LPC device ID. Because these changes reverse all changes in seabios, you no longer need the bios options in the qemu-command-line. instead use the qemu built in seabios (default).

qemu/include/hw/pci/pci_ids.h

    #define PCI_DEVICE_ID_INTEL_ICH9_6    0x2930
    #define PCI_DEVICE_ID_INTEL_ICH9_8    0x2918
    #define PCI_DEVICE_ID_INTEL_Q35_MCH   0x29c0

qemu/hw/isa/lpc_ich9.c

    k->vendor_id = PCI_VENDOR_ID_INTEL;
    k->device_id = PCI_DEVICE_ID_INTEL_ICH9_8;
    k->revision = ICH9_A2_LPC_REVISION;

qemu/hw/pci-host/q35.c

    k->vendor_id = PCI_VENDOR_ID_INTEL;
    k->device_id = PCI_DEVICE_ID_INTEL_Q35_MCH;
    k->revision = MCH_HOST_BRIDGE_REVISION_DEFAULT;

seabios/src/fw/dev-q35.h

    #define PCI_DEVICE_ID_INTEL_ICH9_SMBUS    0x2930
    #define PCI_DEVICE_ID_INTEL_ICH9_LPC      0x2918
    #define PCI_DEVICE_ID_INTEL_Q35_MCH       0x29c0


 Bios
==========

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
    $ git apply qemu_igd.patch
    $ ./configure --disable-werror
    $ make
    $ cd ..
    
e.g. (archlinux)

    $ ./configure --prefix=/usr --sysconfdir=/etc --python=/usr/bin/python2 --smbd=/usr/bin/smbd --enable-docs --libexecdir=/usr/lib/qemu --audio-drv-list=alsa,sdl,pa --enable-gtk --with-gtkabi=3.0 --enable-linux-aio --enable-seccomp --enable-spice --disable-werror --enable-debug

seabios (Un-modified guests)
-------

    $ git clone git://git.qemu.org/seabios.git
    $ cd seabios
    $ git apply seabios_igd.patch
    $ make
    $ mkdir ../biosdir
    $ cp ../qemu/pc-bios/*.bin ../biosdir
    $ cp out/bios.bin ../biosdir
    $ cp out/src/fw/*.aml ../biosdir

linux (Modified Guests)
-----

    $ git clone https://github.com/torvalds/linux.git
    $ cd linux
    $ git apply linux_i915-q35-pch.patch
    $ make mrproper
    $ gunzip -c /proc/config.gz > .config
    $ make
    $ sudo make modules_install
    $ sudo cp -v arch/x86/boot/bzImage /boot/vmlinuz-linux-guest

(archlinux)

    $ sudo mkinitcpio -k <KERNEL NAME> -c /etc/mkinitcpio.conf -g /boot/initramfs-linux-guest.img 
    


