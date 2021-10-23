# Getting started

## Status

Plan 9 is available for general use. 

## System requirements

* 90 MHz or greater Intel/ARM 32-bit processor
* 32 MiB of RAM (system memory for physical and viritualized installs)
* SVGA capable of 800x600 screen resolution or greater
* Network connection
* 300 MiB of storage (SD, IDE, SATA, SAS)
* Either a CD/DVD drive or a USB port for booting the installer media
``` .. note::
These requirements are for the installation procedure. A machine with 16MB 
of memory and no free disk space will work as a terminal but won't run the 
installer so you need a file server and auth server first.

If you wish to install from local media, you need a FAT file system or a 
CD-ROM drive with write capability to get the distribution on a CD. 

If you wish to install over the internet, you need a supported Ethernet card 
or a PPP dial-up account using a modem (not a Winmodem). 
```
### Site Planning

There is nothing magical about installing Plan 9. It's just a matter of populating a Plan 9 file system (typically a fossil file system) and arranging a bootstrap to eventually load a Plan 9 kernel that can use that file system as its root.

Once that initial file server is running, enabling authentication and pxe booting (bootp and tftp) will allow all other local Plan 9 systems to load kernels from the file server and share its file system.

If you find yourself reinstalling Plan 9 frequently, something is wrong. This should not be necessary. In particular, there is no need to give each Plan 9 system its own file system.

### Tested hardware

Plan 9 is known to boot to a graphical desktop on the following machines. Auxiliary functionality such as wireless networking, sound over HDMI, sleep, graphics acceleration, etc. has not yet been tested systematically.

Please contact us if you would like to sponsor the project with a hardware donation. We are especially looking for Lenovo devices from the previous generations that should be available second-hand inexpensively.


### Networking hardware

Not all networking devices are supported by Plan 9 yet. 


### Virtualization environments

``` .. note::
We recommend beginners run Plan 9 on virutual environments if possible. 
This should give you the best hardware support.
```

Users have reported success in running Plan 9 in the following virtualization environments:

* VirtualBox host (on FreeBSD and macOS), known to work in BIOS mode

* VMware host (on Windows), possibly only working in BIOS mode?

* QEMU host (on Linux), works in BIOS mode.


#### QEMU

Create an 1 GiB (or larger) `plan9.img` image file on which you can install the system:

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

The **Plan 9** ISO image is available for download [here](https://https://plan9.io/plan9/download.html).

