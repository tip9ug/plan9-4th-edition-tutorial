# Getting started

## Status

Plan 9 is available for general use. 

## System requirements

* 100 MHz or greater Intel/ARM 32-bit processor
* 512 MiB RAM (system memory for physical and viritualized installs)
* SVGA capable of 800x600 screen resolution or greater
* Network connection
* 500 MiB of storage
* Either a CD/DVD drive or a USB port for booting the installer media


### Tested hardware

Plan 9 is known to boot to a graphical desktop on the following machines. Auxiliary functionality such as wireless networking, sound over HDMI, sleep, graphics acceleration, etc. has not yet been tested systematically.

Please contact us if you would like to sponsor the project with a hardware donation. We are especially looking for Lenovo devices from the previous generations that should be available second-hand inexpensively.


### Networking hardware

Not all networking devices are supported by Plan 9 yet. In those cases, you may want to consider using a USB based networking devices. Plan 9 developers currently have access to the following USB based networking devices which are known to work:

* [USB 802.11n WLAN Adapters based on `ID 0bda:8176 Realtek Semiconductor Corp. RTL8188CUS`](https://vermaden.wordpress.com/2020/10/30/realtek-usb-wifi-review/)
* [USB Wired Ethernet Adapters based on `ID 0b95:772b ASIX Electronics Corp. AX88772B`](https://www.freebsd.org/cgi/man.cgi?query=axe)

### Virtualization environments

``` .. note::
    We recommend running Plan 9 on real hardware ("bare metal") if possible. This should give you the best possible performance and hardware support.
```

Users have reported success in running Plan 9 in the following virtualization environments:

* VirtualBox host (on FreeBSD and macOS), known to work in BIOS mode

* VMware host (on Windows), possibly only working in BIOS mode?

* QEMU host (on Linux), works in BIOS mode.


#### QEMU

Create an 1 GiB (or larger) `ghostbsd.img` image file on which you can install the system:

```
$ pwd
/home/user
$ mkdir -p .qemu/plan9
$ fallocate -l $(( 1*1024*1024*1024 )) .qemu/plan9/plan9.img
```

Then, boot Plan 9:

```
qemu-system-x86_64 -machine type=q35,accel=kvm \
-enable-kvm -cpu host -smp 2 -m 1024 \
-device virtio-net,netdev=vmnic -netdev user,id=vmnic,hostfwd=tcp::5222-:22 \
-vga std -soundhw hda -no-quit \
-drive format=raw,file=${HOME}/.qemu/plan9/plan9.img \
-drive format=raw,file=${HOME}/Downloads/plan9.iso \
-boot menu=on
```

When QEMU starts, press `esc` and select `2` to boot the ISO.





## Downloading

The **Plan 9** ISO image is available for download [here](http://ghostbsd.org/download).

