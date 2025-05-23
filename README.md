GRUB2 Live ISO Multiboot
========================

This version: https://github.com/eugenesan/glim is forked from: https://github.com/cshandley-uk/bash_glim
Original (now abandonned) project can be found at:
https://github.com/thias/glim or http://glee.thias.es/GLIM


Overview
---

GLIM `[G]RUB2 [L]ive [I]SO [M]ultiboot` is a set of grub configuration files
to turn USB storage device containing many GNU/Linux and *BSD distribution ISO images
into a neat device from which many different Live environments and Installation media can be used.
The same can be achieved by unpacking Windwows ISO images onto separate NTFS partitions.

GLIM can be viewed as a more basic but completely open source alternative to Ventoy.
In fact, GLIM provides only config files while runtime binaries are provided by GRUB2 on the host.

Advantages over extracting files or using special Live USB creation tools :

 * A single USB memory can hold all Live environments (the limit is its size)
 * ISO images stay available to burn real CDs or DVDs
 * ISO images are quick to manipulate (vs. hundreds+ files)

Disadvantages :

 * There is no persistence overlay for distributions which normally support it
 * Setting up isn't as easy as a simple cat from the ISO image to a block device

As modern Linux ISOs often exceed the 4GB file size limit of FAT32, GLIM now 
supports a second partition using other filesystems supported by GRUB2, such as
ext3/ext4, F2FS, NTFS or exFAT - but the distribution must also support booting from
it, which isn't the case for many with NTFS (Ubuntu does, Fedora doesn't),
F2FS (Ubuntu doesn't) and exFAT (Ubuntu doesn't, Fedora does).
Ext4 is a safe bet for the second partition.
Note: Writing to Ext4 on some flash drives can be extreamely slow,
try adding `-O sparse_super2,^has_journal,^uninit_bg,^ext_attr,^huge_file,^64bit` when formatting.


Screenshots
---

![Main Menu](https://github.com/thias/glim/raw/master/screenshots/GLIM-3.0-shot1.png)
![Ubuntu Submenu](https://github.com/thias/glim/raw/master/screenshots/GLIM-3.0-shot2.png)


Recent changes
---

* GLIM now easily supports ISO files >4GB through the use of a second partition,
although you can still use a single partition if you want.

* The ISO folder has been moved from `/boot/iso/` to just `/iso/`, so that it's
easier to find, and also is in the same location whether you use one or two partitions.
GLIM will search for '/iso' according to partitions order and use the first one found.

* Glim now supports booting Windows (x64) 10+ Setup
or Preinstall Environment from separate NTFS/FAT/ExFAT partitions.

* Since vast majority of ISOs are uniquely named, only openbsd and calculate ISOs
should be placed in their respective directories.
The rest of the ISOs are expected to reside in `/iso/`.

* Added the `format_empty_disk.sh` script.

Requirements / Partitioning
---

You need a USB memory stick (or external hard drive!) partitioned & formatted 
one of the following ways:

1. A single partition formatted as FAT32 with the filesystem label `GLIM`. 
You can use GPT or MBR as needed but GPT is preffered unless legacy BIOS support is desired.

2. Two partitions. The small first partition must be formatted as FAT32 with
the filesystem label `GLIM` and recommended size of 32MB (actual GLIM size is only 12MB).
The second partition should be formatted as Ext4 with the filesystem label `GLIMISO`.
It's best if the USB stick uses MBR, but if it uses GPT (as GNOME's Disks utility does) then
GRUB only supports installing for EFI (not BIOS) - unless you add a third BIOS Boot partition.
GLIM needs the BIOS Boot partition to come after the other two partitions.
See notes below regarding BIOS boot partition.

3. Optionally you can add more paritions. For example:
 * Add generic partition for file transfers. It is recommended to format it as ExFAT.
 * Add LUKS/Ext4 parition to use with Linux LiveDVDs and Tails.
   To trick Tails to use this partition as Persistent storage:
    1. Name the LUKS container and the ext4 partition as "TailData"
    2. Change partitiontype to "Linux Reserved"
    3. Create folders: `dont-ask-again`, `Persistent/Tor Browser` owned by user 1000:1000
    4. Add `/home/amnesia/Persistent	source=Persistent` to persistence.conf owned by user 115:122
    Note:
     You might see a harmless warning regarding failed Persistent storage upgrade.
     It happens because Tails assumes the Tails image is directly on the USB drive
     when Persistent storage is available.
 * Add NTFS/FAT/ExFAT formatted paritions with the content of Windows Setup or Windows LiveCD ISOs.
   Windows boot entry will be added for each detected partition.
   Optionally you can place autounattend.xml on those partitions if you want to customize the installation process.
   To create your own autounattend.xml visit: https://schneegans.de/windows/unattend-generator/

   Note:
    If you want to be able to mount one of the partitions on Android/Windows
    it is recommended to format it as FAT32 or ExFAT and place it as a first partition.
    To do that you need to disable CHECK_ORDER variable in glim.sh prior to installation
    to allow non-default partitions order.
    Unsupported partitions migh result in anoying but harmless pop-ups/warnings.

 Here is an example of a disk layout that covers all supported use cases:

```
Disk /dev/sda: 124GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Number  Start   End     Size    File system  Name       Flags             Purpose
1      1049kB  64.4GB  64.4GB  exfat        GLIMXFAT   msftdata          [Generic file storage accesible on Linux/Windows/Andoird/MacOS]
3      64.4GB  98.8GB  34.4GB  ext4         GLIMISO                      [ISO storage and Generic Storage for Linux]
4      98.8GB  107GB   8594MB  luks+ext4    TailsData                    [Encrypted Persistent Storage for Tails]
5      107GB   116GB   8403MB  exfat        GLIMWIN11  msftdata          [Windows 11 Setup]
6      116GB   124GB   8401MB  ntfs         GLIMWINPE  msftdata          [Windows 11 PE HBCD]
2      124GB   124GB   33.6MB  fat16        GLIM       boot, hidden, esp [GLIM boot partition]
```

Notes:
See the following link for details on how to create a BIOS Boot partition: 
https://wiki.archlinux.org/title/GRUB#GUID_Partition_Table_(GPT)_specific_instructions
Basically create an unformatted 1MB partition at the end of the disk, then 
change it's partition type to "BIOS Boot" (which has the 
GUID `21686148-6449-6E6F-744E-656564454649`).  You can do this with GNOME's 
Disks utility, without resorting to the terminal!

If you find partitioning difficult, then you can try using the new experimental 
script `format_empty_disk.sh`, which will ask you a few questions before
setting-up an empty disk with GLIM's recommended two partition GPT set-up 
(plus a BIOS Boot partition), ready to use with `glim.sh` itself.
Using the sript should be safe, so for example it shouldn't delete any partitions,
only create new ones... And in the event of any errors, the script should stop rather than risk 
doing anything wrong.
HOWEVER, it is still a very new script, which likely contains bugs,
so please open issues about any problems you experience.
You use this script entirely at your own risk. If it formats your entire computer, then that is your problem.
So please make sure you have a recent backup before using it or better yet,
use LiveCD with permanent storage disconnected/disabled.


Installation
---

Mount the GLIM partition (and the GLIMISO partition if present) on your USB 
memory stick (or external hard drive).

Then clone the git repository (or use Code > Download ZIP before unzipping it), 
and just run the script (as a normal user) :
```./glim.sh```

Once finished, you may change the filesystem label to anything you like. 
The script will have created an `/iso/` folder, inside of which you will see
a few empty folders for distributions with oddly named ISOs.
The rest if ISOs can be placed directly in `/iso/` folder.

The supported ISOs (in alphabetical order) are:

[//]: # (distro-list-start)

* [`almalinux`](https://almalinux.org/) - _Live Media only_
* [`antix`](https://antixlinux.com/)
* [`arch`](https://archlinux.org/)
* [`artix`](https://artixlinux.org/)
* [`bodhi`](https://www.bodhilinux.com/)
* [`calculate`](https://wiki.calculate-linux.org/desktop)
* ~~[`centos`](https://www.centos.org/)~~ - _Live was discontinued_
* [`clonezilla`](https://clonezilla.org/)
* [`debian`](https://www.debian.org/CD/live/) - _live, mini & partial netinst_
* [`elementary`](https://elementary.io/)
* [`fedora`](https://fedoraproject.org/)
* [`finnix`](https://www.finnix.org/)
* [`gentoo`](https://www.gentoo.org/)
* [`gparted`](https://gparted.org/)
* [`grml`](https://grml.org/)
* [`ipxe`](https://ipxe.org/) - _.iso or .efi_
* [`kali`](https://www.kali.org/)
* [`kubuntu`](https://kubuntu.org/)
* [`libreelec`](https://libreelec.tv/)
* [`linuxmint`](https://linuxmint.com/)
* [`lubuntu`](https://lubuntu.me/)
* [`manjaro`](https://manjaro.org/)
* [`memtest`](https://memtest.org/) - _Only .bin/.efi, not .iso_
* [`mxlinux`](https://mxlinux.org/)
* [`netrunner`](https://www.netrunner.com/)
* [`openbsd`](https://www.openbsd.org/)
* [`opensuse`](https://www.opensuse.org/) - _Live from Alternative Downloads only_
* [`peppermint`](https://peppermintos.com/)
* [`popos`](https://pop.system76.com/)
* [`porteus`](http://www.porteus.org/)
* [`rescuezilla`](https://rescuezilla.com/)
* [`rhel`](https://www.redhat.com/rhel) - _Installation only_
* [`rockylinux`](https://rockylinux.org/)
* [`slitaz`](https://slitaz.org/)
* [`supergrub2disk`](https://www.supergrubdisk.org/)
* [`systemrescue`](https://www.system-rescue.org/)
* [`tails`](https://tails.net/)
* [`ubuntubudgie`](https://ubuntubudgie.org/)
* [`ubuntu`](https://ubuntu.com/)
* [`void`](https://voidlinux.org/)
* [`xubuntu`](https://xubuntu.org/)
* [`zorinos`](https://zorin.com/os/)

[//]: # (distro-list-end)

Any unpopulated/unsupported folder and unsupported ISOs will be ignored.
To disable any ISO just move it to unsupported folder or add one or more charachters
in the beginning of its filename.

Download the right ISO image(s) to the `/iso/` or dedicated directory. If you require
boot parameter tweaks, edit the appropriate `boot/grub2/inc-*.cfg` file.

Items order in the menu
---

Menu items for a distro are ordered by modification time of the iso files
starting from the most recent ones. If some iso files have the same mtime, their
menu items are ordered alphabetically.

Here is a generic idea how to keep it nicely ordered when you have multiple
releases of some distro:

* touch your **release** iso files with the release date
* touch your **point release** iso files with the original release date plus a
  day per point. This is a way to ensure point releases never pop above the next
  release like Debian 10.13.0 (released 10 Sep 2022) would still be below Debian
  11.0.0 (released 14 August 2021)
* in case there are multiple flavours of some iso but the version is the same,
  touch all of them with the same date for the whole group to be ordered
  alphabetically

Sample ordered menu:

|                                    | iso mtime               |
|------------------------------------|-------------------------|
| Debian Live 12.0.0 amd64 standard  | 10 June 2023            |
| Debian Live 11.7.0 amd64 gnome     | 14 August 2021 + 7 days |
| Debian Live 11.7.0 amd64 kde       | 14 August 2021 + 7 days |
| Debian Live 11.7.0 amd64 standard  | 14 August 2021 + 7 days |
| Debian Live 11.0.0 amd64 gnome     | 14 August 2021          |
| Debian Live 11.0.0 amd64 kde       | 14 August 2021          |
| Debian Live 11.0.0 amd64 standard  | 14 August 2021          |
| Debian Live 10.13.0 amd64 standard | 6 July 2019 + 13 days   |
| Debian Live 9.13.0 amd64 standard  | 17 June 2017 + 13 days  |

Special Cases
---

### iPXE

The `.iso` files don't work when booting using EFI, you simply need to use
`.efi` files instead.

### LibreELEC

LibreELEC isn't provided as ISO images, nor is it able to find the `KERNEL` and
`SYSTEM` files it needs anywhere else than at the root of a filesystem.
But it's useful to enable booting the installer by just copying both
files to the root of the USB memory stick.
Live booting is also supported, and the first launch will create a 512MB file
as /STORAGE.

### Memtest86+

The `.iso` file doesn't work. Use either the `.bin` or the `.efi` depending on
the boot mode used.

### Ubuntu

Recent Ubuntu desktop iso images bundle multiple versions on the Nvidia
driver. With that, the images are over 4GB, the FAT32 max file size. For example
`ubuntu-20.04.6-desktop-amd64.iso` is 4.1GB, `ubuntu-22.04.2-desktop-amd64.iso`
is 4.6GB. The driver is not required in a live system, it can be removed to make
an image fit into 4GB. For example, with 22.04.2 image in the current dir:

```
mkdir slim
iso=ubuntu-22.04.2-desktop-amd64.iso

xorriso -indev "$iso" -outdev slim/"$iso" \
    -boot_image any replay -rm_r /pool/restricted/{l,n} --
```

Now you can copy `slim/ubuntu-22.04.2-desktop-amd64.iso` to your FAT32 formatted
GLIM USB stick.

Some Ubuntu flavours also bundle the Nvidia driver (like Kubuntu), some don't
(like Xubuntu). The same trick can be used with the former.


Testing
---

With KVM it should "just work". The `/dev/sdx` device should be configured as
an IDE or SATA disk (for some reason, as USB disk didn't work for me on Fedora
17), that way you can easily and quickly test changes.
Make sure you unmount the disk from the host OS before you start the KVM
virtual machine that uses it.
For UEFI testing, you'll need to use one of the `/usr/share/edk2/ovmf/*.fd`
firmwares.


Troubleshooting
---

If you have any problem to boot, for instance stuck at the GRUB prompt before
the menu, try re-installing.
If you have other exotic GRUB errors, such as garbage text read instead of the
configuration directives, try re-formatting your USB memory from scratch.
I've seen weird things happen...


Contributing
---

If you find GLIM useful but the configuration of the OS you require is missing
or simply outdated, please feel free to contribute! What you will need is to
create a GitHub pull request which includes :
 * All changes properly and fully tested.
 * New entries added similarly to the existing ones :
   * In alphabetical order.
   * With all possible variants supported (i.e. not just the one spin you want).
 * An original icon of high quality, and a shrunk 24x24 png version. Using
   `convert -size 24x24 -background 'rgba(0,0,0,0)' original.svg small.png`
   may work.
 * An updated supported directories list in this README file.


Credits
---

* Copyleft 2012-2023 Matthias Saou http://matthias.saou.eu/
* Copyleft 2025 Chris Handley https://github.com/cshandley-uk
* Copyleft 2025 Eugene Sanivsky (eugenesan) https://github.com/eugenesan

All configuration files included are public domain. Do what you want with them.
The invader logo was made by me, so unless the exact shape is covered by
copyright somewhere, do what you want with it.
The background is "Wallpaper grey" © 2008 payalnic (DeviantArt)
The `ascii.pf2` font comes from GRUB, which is GPLv3+ licensed. For more
details as well as the source code, see http://www.gnu.org/software/grub/
