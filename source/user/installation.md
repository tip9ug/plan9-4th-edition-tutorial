## Installation Procedure


### Boot the installation CD

(NOTE on vmware fusion for Mac OS X, you probably want to use SCSI disks as the IDE emulation makes Plan 9 difficult to boot at this time. SCSI on the other hand works VERY fast for booting and installtion)

Cold boot (power off, power on) your machine using your CD. You will be given the choice of installing Plan 9 or just booting a full Plan 9 system directly from the CD.

Booting the system directly lets you explore the system without installing and also makes a good recovery CD.

The installer on the CD assumes that your CD-ROM is on the second IDE master. If your CD-ROM is elsewhere, you will see the following error:

Unknown boot device: sdD0!cdboot!9pcflop.gz
Boot device: fd0
boot from:

IDE/ATAPI drives in Plan 9 are named like sdD0, where the capital letter is C for the primary IDE interface and D for the second. The digit is 0 for the master and 1 for the slave. So if your CD-ROM is the primary slave, use sdC1:

boot from: sdC1!cdboot!9pcflop.gz

If you find yourself at a "boot from:" prompt or a "root is from:" prompt, it is likely that the bootstrap program has not detected your floppy drive. See installation troubleshooting.

You will be asked if DMA should be enabled for your ide drives, the default is 'yes', only answer 'no' if you had problems during the installation with DMA enabled.

You will be prompted for a "mouseport"; if you have a USB mouse ignore this prompt, you won't have mouse support during install, but that should be OK.

Last you will be asked for your vga settings, the default resolution is the safest option and should be enough to run the installer, you can change this later once the system is installed.

Note: If you know your videocard is not supported, answer "vesa" when prompted for a monitor type.

### Begin the installation

If your video card and resolution is supported (or you selected vesa) the window system will start.

Otherwise you will be dropped into a % prompt. To start the installer in text mode run:

```
% inst/textonly
```

The installer is the same in text and graphical mode, but graphical mode is a bit more friendly.

In graphical mode the window system (rio(1)) will display a grey screen with some windows. The large upper window contains the install process itself. The window under it, is a running log of what has happened. A statistics graph is in the bottom corner; the graph, from top to bottom, shows system load, memory usage, interrupt rate, system call rate, context switch rate, and ethernet packet rate.

Interaction with the installation program is textual and you don't need to interact with rio during the install process. For systems with very small screens, you may find necessary to use the keyboard's arrow keys to scroll the window up and down.

When you are prompted to provide information (e.g., an IP address) or asked to select from a list of choices (e.g., the disk to use), the prompt will be of the form

```
	Your choice (foo, bar, baz, quux)[quux]:
```

When there is a small list of possible answers, they will be listed in parentheses. If there is a default choice, it will be given in square brackets and pressing enter will select this default.

The installation program is structured as a sequence of tasks that must be performed; some tasks have successful completion of others as prerequisites.

At each step, you will be shown the list of completed tasks and the list of tasks that are ready to be done. Task names appear in parentheses in the text that follows. Typing <control-d> at any prompt will abort the current task and return you to the main menu.

### Choose a filesystem

(Configfs) Fossil(4) is the Plan 9 fileserver. Venti(8) is an archival block storage server. You may run fossil on its own or as a write buffer backed by a Venti server. The primary value of using Venti is to store daily snapshots of your filesystem (see yesterday(1)).

### Partition your Hard Disk Drive (HDD)
  
(Partdisk) First you need to setup a partition for Plan 9. If you want to boot Plan 9 directly or via a boot loader like LILO or the Windows boot menu, you need to allocate a primary partition. If you are content to boot from a floppy disk or from DOS via ld.com (see 9load(8)), you can use a secondary partition.

The install process will scan your disk devices and give you a list of them, along with manufacturer identification strings and the disks' partitions tables. For example:

The following disk devices were found.

```
sdC0 - WDC AC36400L
 * p1                     0 2709        (2709 cylinders, 19.53 GB) FAT32LBA
   empty               2709 3266        (557 cylinders, 4.01 GB)
   p3                  3266 3807        (541 cylinders, 3.90 GB) BSD386
   s4                  3807 4367        (560 cylinders, 4.03 GB) LINUX
   s5                  4367 4368        (1 cylinders, 7.38 MB) LINUXSWAP
   empty               4368 6201        (1833 cylinders, 13.21 GB) empty

sdD0 - IDE-CD ReWritable-2x2x63.014VO07982013140700210

Disk to partition (sdC0, sdD0)[no default]:
```
  
The Plan 9 names for storage devices have the form sdXX. The names sdC0 and sdC1 are the master and slave on the primary ATA controller; sdD0 and sdD1 are on the secondary, and sdE0, sdE1, sdF0, and sdF1 are on additional ATA cards. SCSI devices are named sdNT, where N is the SCSI controller number and T is the SCSI target number.

Once you have chosen the disk, you will need to create a Plan 9 partition. To do this, the install process will run the Plan 9 fdisk program and let you partition the disk. If the disk does not already have a Plan 9 partition, fdisk will suggest one by creating (but not writing) a partition in the largest contiguous empty space it can find. For example, you might see:

```  
cylinder = 7741440 bytes
 * p1                     0 2709        (2709 cylinders, 19.53 GB) FAT32LBA
 ' p2                  2709 3266        (557 cylinders, 4.01 GB) PLAN9
   p3                  3266 3807        (541 cylinders, 3.90 GB) BSD386
   s4                  3807 4367        (560 cylinders, 4.03 GB) LINUX
   s5                  4367 4368        (1 cylinders, 7.38 MB) LINUXSWAP
   empty               4368 6201        (1833 cylinders, 13.21 GB) empty
>>>
```
  
Each line contains a partition name (p1, p2, p3, and p4 are the only valid primary names; s1, etc. are the only valid secondary names).

Fdisk also shows the starting and ending cylinder, the size of the partition, and the type of partition. Note that partitions include the starting cylinder but not the ending cylinder. >>> is the prompt. Typing h or ? at this prompt will print the help message. In this example, the * next to p1 means that p1 is the active partition (i.e. the one used when booting from the disk), and the single quote (') next to p2 means that the partition table entry for p2 is different from what is on the disk; that is, changes have been made but not written. In this example, fdisk has created p2 in what was previously unpartitioned space.

If you agree with fdisks' proposal, you need only type w to write the changes and then q to quit fdisk. Otherwise, you can edit the table yourself, using the "a pN" and "d pN" commands to add and delete partitions.

Create the Plan 9 partition and quit fdisk.

See prep(8) for more information on using fdisk.

### Prepare the Plan 9 Partition

(Prepdisk) Plan 9 partitions are further subdivided into named partitions. You need to use disk/prep to create partitions named 9fat, fossil, swap and if you selected fossil+venti arenas and isect, disk/prep will suggest a sensible layout.

Note that 9fat must be at the beginning of the Plan 9 partition in order to boot.

See prep(8) for more information on using prep.

### Format Fossil

(fmtfossil) You will be prompted for the fossil partition to format, which should have been created in the previous step.

### Choose and mount the file system to install on

(Mountfs) You will be prompted to choose a fossil partition to install onto (and which will hold your data). The default should be the partition you formatted in the previous step.

### Choose how to obtain the distributon archive

(Configdist) Now you have to select where the distribution archive will be obtained from: local media or the network. If you are installing from the CD select 'local' and skip ahead to the next section.

If you choose the network, you will be stepped through setting up an ethernet or PPP connection and then downloading the archive.

If you are using Ethernet, you can enter your IP configuration manually or via DHCP. If you choose to enter the configuration manually, be sure to have your IP address, network mask, and gateway IP address.

If you are using PPP, you will have to choose a serial device and connection method. You can choose to dial and log yourself in or to have Plan 9 dial and use CHAP to log in (this is how the stock Windows PPP client connects, for example).

If you choose to log yourself in, you will be dropped into a conversation with the modem. Dial, log in, and once PPP has started, type <control-d>. You may need to type <control-m> rather than <enter> to get a response from the modem.

If you use CHAP, the install process will prompt for a phone number (exactly as you would dial it yourself, with any necessary prefixes; numbers only), user name, and password. It will then dial and initialize the connection.

(Download) Once you have a network connection, run the download task to download the archive from http://plan9.bell-labs.com to the file system mounted in the last step, in the directory /dist. If the download is interrupted and you start it again, it will pick up where it left off. If you restart the installation program after downloading the archive, you may need to tell mountdist where it is.

Once the download is complete, you may wish to run the task stopppp to hang up your PPP connection. Similarly, stopether will deactivate your Ethernet connection.

### Locate and mount the distribution archive

(Mountdist) If you downloaded the archive, the install program has written it to /dist on your chosen partition. The install program will run mountdist with this information for you, so you can skip to the next step.

Otherwise, mountdist presents you with a list of FAT (DOS, Windows), ISO-9660 (CD-ROM), and Plan 9 file systems that it can read. You must choose a file system and then point out the directory containing the archive.

The archive may be in one of three forms:

 -  A bzipped CD image named plan9.iso.bz2. This is the file you obtain from the download page on the Bell Labs server.
 -  A CD image named plan9.iso. This is the result of uncompressing plan9.iso.bz2. If you store the uncompressed image on a FAT file system beforehand, then the install program will not need to uncompress it, which will save disk space on the file system.
 -  The contents of the CD image, in a directory tree. This is the CD image itself, typically written to a CD.

When prompted for "Distribution disk" the usual value is /dev/sdD0/data (but your CD drive may be a different sdXX).

Once you have chosen a file system, you need to point out the directory containing the archive. Type a slash-separated path name relative to the root of the chosen file system. If you type "browse" instead of a directory name, you will be dropped into a minimal shell that you can use to find the files. Specifically, the shell has three commands: "cd dir"; changes directories, "lc"; prints a columnated list of files for the current directory, and "exit". Once you are in the directory containing the archive (or if you give up the search), exit the shell.


### Format Venti (Optional)

(fmtventi) If you selected fossil+venti you will now be prompted for the arenas and index (isect) partitions usually created during the 'prepdisk' step, in that case the default values should be all you need.

Note that this step can take a long time with slow disks or qemu.

### Copy the archive to the file system

(Copydist) Once the archive has been located or downloaded, selecting unpack will extract the distribution archive to the newly created fossil file system. The log window will display the name and size of each file as it is extracted. This takes anywhere from 10 minutes to an hour depending on the speed of your computer and disks.


### Configure one or more ways to boot Plan 9

(Bootsetup) The first time you run bootsetup, it initializes the 9fat configuration partition with the appropriate bootstrap code as well as a modified version of your plan9.ini from the boot floppy or CD and a 9pcf kernel.

In order to boot into Plan 9, another bootstrap program must locate this partition, read plan9.ini, and boot the kernel. There are a number of ways to make this happen, all selectable from the bootsetup menu. If you wish to use more than one method, simply run bootsetup multiple times.

The boot methods are:

 -  floppy. Create a boot floppy. In addition to a bootstrap program, the floppy will contain a kernel and a backup of your plan9.ini file named plan9ini.bak, but will not use them. Instead, the floppy will load plan9.ini and the kernel from your 9fat partition. To boot the kernel on the floppy (useful as a rescue mechanism if you trash your 9fat partition), copy plan9ini.bak to plan9.ini and change the line bootfile=sdXX!9fat!9pcdisk to bootfile=fd0!9pcdisk.gz.

 -  plan9. Set the Plan 9 partition to be the active one (i.e. the partition booted by default). This is only useful if you have installed Plan 9 on your first hard disk (for IDE systems, sdC0). You can always set another partition active later by using disk/fdisk.

 -  win9x. Edit the Windows startup menu to list Plan 9 as an option. Your c:\config.sys and c:\autoexec.bat files will be saved as config.p9 and autoexec.p9, and then edited. A bootstrap program as well as plan9ini.bak and a kernel will be copied to the directory c:\plan9 (created if necessary). The procedure described above for rescue works here too, but the bootfile should become sdC0!dos!plan9/9pcdisk. This may or may not work with Windows Me. We have not tried it.

 -  winnt. Edit the Windows NT/2000/XP boot menu to list Plan 9 as an option. This is only possible when your ``c:'' drive is a FAT partition, since the boot configuration must be accessible. Your c:\boot.ini file will be saved as boot.p9, and then edited. This will also create the file c:\bootsect.p9, which the boot manager will use to load Plan 9.

### Finish

(Finish) Choosing the finish task will halt the file system and print a message saying it is safe to reboot your computer.
