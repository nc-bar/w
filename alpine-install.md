# Alpine linux with encryption

Alpine is a linux distribution focused on security, simplicity and efficiency. According to the project's webpage:

> Alpine Linux is an independent, non-commercial, general purpose Linux distribution designed for power users who appreciate security, simplicity and resource efficiency.

They take some design decisions which are very interesting. For example, the `c` standard library used is [musl](https://www.musl.libc.org) instead of glibc. Also, the init system is not systemd but openRC. The usual unix tools (`ls`, `grep`, etc.) are provided by the busybox binary, used in embedded systems.

The initial installation comes with a very reduced set of programs. Among them is the `apk` package manager, which can be used to install precompiled software and take care of the dependencies automatically. The minimal initial system and the `apk` manager are great for building a minimal system.


## Initial configurations

An iso image can be downloaded from the project's [website](https://alpinelinux.org/downloads) and then copied into a usb stick or a dvd.

After booting, a login screen appears. Entering root will log you in and typing `setup-alpine` launches the installation manager. The installation is fairly straight-forward if no special configurations are needed.

After the keyboard, root password, timezones and network configuration, the installation script will prepare the disk. At that moment, if you want to use encryption, quit the script.


## Encryption

The objective is to have an encrypted partition inside which every sensitive information can be stored. The encryption and decryption have to be transparent for the user and all the programs: typing a password to open or save every file is too troublesome. Ideally the system will encrypt itself at shutdown and ask for a password at boot.

The solution is to use [dm-crypt](https://wiki.archlinux.org/index.php/Dm-crypt). This kernel module can encrypt and decrypt a transparently, providing a virtual device which can be used as if it were a physical one. The device appears under `/dev/mapper/`.

I chose the following disk layout:

	+-------------------------------+
	|  /sda1  |  /sda2              |
	| (/boot) |  encrypted lvm      | 
	+-------------------------------+

The `/boot` partition contains the necessary files to boot, such as the kernel binary, so it needs to be unencrypted. It's possible to encrypt `/boot` as well, but it requires support from the bootloader.

The second partition is encrypted and will contain the whole system, except for the files in `/boot`.

The [fdisk](https://wiki.archlinux.org/index.php/Fdisk) tool can be used to create the partition scheme.

The `cryptsetup` utility talks to `dm-crypt`. It doesn't come with the installation image, so use `apk` to add it:

	apk add cryptsetup

Once installed, assuming your storage device is in `/dev/sda`, the command

	cryptsetup luksFormat /dev/sda2

will format the `/dev/sda2` partition with the LUKS format, which is used by `dm-crypt`. It's possible to use a keyfile instead of a password, but I decided to keep it simple instead.

Make sure to backup the LUKS header in that partition (if it breaks you will lose the information inside said partition) using

	cryptsetup luksHeaderBackup /dev/sda2 --header-backup-file luks_backup_alpine_sda2

Then, open it. This means telling the kernel to create a mapped device which can be used as if it were a physical one. The command

	cryptsetup open /dev/sda2 crypto

will tell the kernel to create the `/dev/mapper/crypto` device to interact with the encrypted partition transparently.

At this point the file system can be created, and installation resumed, but I chose to configure lvm on top of `dm-crypt`, because I wanted to understand how it works.


## Logical volume management

[Lvm](https://wiki.archlinux.org/index.php/LVM) is used to make partition management more flexible. It solves the problem of wanting to shrink or grow a partition, without having enough contiguous space.

	file system <--> lvm <--> dm-crypt <--> drivers <--> storage

Just as `dm-crypt` provides a device other software can talk to, while keeping it's operations hidden, lvm does the same, but mapping blocks in the exposed device to blocks in the underlying device. This makes it possible to have a partition spread over non contiguos blocks of storage, which in turn makes the partition scheme more flexible.

The concepts to understand here are: physical group, volume group and logical volume. The physical group is the actual device; it could be a disk, a partition, or any block device interface provided by the kernel, such as a `dm-crypt` mapped device or a loopback device from a file. The volume group is the set of physical groups used to store the logical volumes; this gives flexibility to lvm: the logical volumes can be located in different devices. The logical volumes are the analog of partitions; they appear as devices that can be mounted, usually under `/dev/vg/name` where `name` is it's name, and `vg` is the name of the related volume group.

I created the volume group using the `dm-crypt` backed device as a physical volume. Then, I created logical volumes for `/`, `/home` and `swap` without using the entire disk, in case I want to experiment.

To set up lvm, the `lvm2` package is needed, run `apk add lvm2` to get it. Then, configure a device as a physical group

	pvcreate /dev/mapper/crypto

Then define a volume group with the previous device

	vgcreate vg0 /dev/mapper/crypto

The name of the volume group will be `vg0`. Now, you can add the logical volumes

	lvcreate -n swap -L 4G vg0
	lvcreate -n home -L 100G vg0
	lvcreate -n root -L 80G vg0

The `-n` flag sets the name and the `-L` flag the size. To activate the volume group `vg0` execute

	vgchange -a n vg0

Now, the partitions will be available under `/dev/vg0/partition_name`.

After using the system for a while I didn't noticed performance issues brought by stacking lvm on top of `dm-crypt`. I didn't see many advantages either, but I did it mostly to learn, so the increased complexity was worth it.


## Creating the file systems

Once the devices are ready, the file systems can be put in place to store the system files. I chose `ext4` for the root partition and `ext2` for boot, mostly because that's what I've allways used and never failed me.

The tools to create the file systems come with the `e2fsprogs` package, so install them using `pkg add e2fsprogs`. Then, create the appropiate file systems, for example

	mkfs.ext4 /dev/vg0/root
	mkfs.ext2 /dev/sda1

Also, format the swapp partition, if any:

	mkswap /dev/vg0/swap

Now that the file systems exist, the installation script can copy the system files it needs. But first, you need to mount them

	mount -t ext4 /dev/vg0/root /mnt
	mount -t ext2 /dev/sda1 /mnt/boot

Finally, the script can proceed and install the base system. The `setup-alpine` script is made up of several scripts, each one solving a particular problem. The one we need now is `setup-disk`, which takes as an optional parameter the device or mountpoint directory used to copy the files into. Call it as `setup-disk -m sys /mnt/`, or wherever `root` is.

After the script is done add the partition to the `/etc/crypttab` file of the installed system, located in `/mnt/etc/crypttab`, assuming you mounted root into `/mnt/`. The line to add is

	crypto	UUID=<uuid>	none	luks

If the file doesn't exist, create it. This file tells the operating system that a luks volume exists with node `/dev/mapper/crypto`. Replace `<uuid>` with he UUID of the partition; to get it run

	blkid -s UUID -o value /dev/sda2 > ~/uuid

which saves the id into a file named uuid in the root home of the live linux (not the new installation), since it'll be needed again later.


## New initial ram disk

The next step needs an introduction to the linux boot process.

At startup, the computer finds a bootable device: a hard-drive, usb-stick, even an ethernet interface for network boot. Then, assuming it's a storage device, it extracts the [Master Boot Record](https://wikipedia.org/wiki/Master_boot_record), or mbr for short; it contains the partition scheme of the device as well as executable code, called the boot loader.

The boot loader is a small piece of code in charge of loading operating system kernels into memory, along with any requiered parameters. Modern boot loaders can be complex and divide its operations into stages, providing more features, but the main goal is to load the operating system.

In linux, a separate partition, usually called boot, contains the files needed to initialize the system, such as the kernel image. The boot loader reads this partition and loads the kernel.

If everything went well, the first process, [init](https://wikipedia.org/wiki/Init), is launched. This process is in charged of mounting the required filesystems, activating the swapp, launching other important processes, etc. It's located in the root file system, which creates a problem if the partition it's in is ecrypted: the kernel will not be able to load it.

A solution is to use the initial ram disk. Along with the kernel, the boot partition contains a file used to set up a temporal filesystem in memory. Inside it, there is an init script used to mount the actual root file system and then give the control to the actual init script. So, if you use encryption, you will need to create a new initial ram disk with the modules to decrypt partitions.

To add `cryptsetup` to the initial ram disk, append `cryptsetup` to the `features` list inside the file `/mnt/etc/mkinitfs/mkinitfs.conf`. The following command generates the initial ram disk:

	mkinitfs -c /mnt/etc/mkinitfs/mkinitfs.conf -b /mnt/ $(ls /mnt/lib/modules)


## Bootloader

There are many bootloaders, I went with [Syslinux](https://wikipedia.org/wiki/SYSLINUX), but [Grub](https://wikipedia.org/wiki/GNU_GRUB) is also a possibility. Make sure the one you choose can read the file system of the `/boot` partition, if it's `ext2` it probably can.

First, let's install the `syslinux` package:

	apk add syslinux

As mentioned before, the bootloader lives in the MBR of the disk and it's task is to load the kernel with the appropiate parameters.

Since we're using encryption, the kernel will need to know which disk needs to be decrypted and mounted as well as it's name. I chose the name `crypto` for `/dev/sda2`, so append the string

	cryptroot=UUID=<uuid> cryptdm=crypto

to the `default_kernel_opts` parameter list, in the file `/mnt/etc/update-extlinux.conf`. The first tuple defines the device with UUID `uuid` as the encripted root file system, so put the UUID of the encrypted root there. The second defines the name crypto as the mapping name and should be the same name used in `/mnt/etc/crypttab`.

The `update-extlinux` utility is used to generate an image with the bootloader, which can then be copied into the mbr. Since the paths it uses are hardcoded, a chroot is needed:

	chroot /mnt
	update-extlinux
	exit

The `update-extlinux` program may return an error, I ignored it and nothing happend. This [alpine guide](https://wiki.alpinelinux.org/wiki/LVM_on_LUKS) advices the same.

Now, you can write the bootloader into the mbr of the device, `/dev/sda` in my case

	dd bs=440 count=1 conv=notrunc if=/mnt/usr/share/syslinux/mbr.bin of=/dev/sda


## Finishing the installation

At this point the installation is complete. Close everything and reboot

	sync
	umount /mnt/boot
	umount /mnt
	vgchange -a n vg0
	cryptsetup close /dev/mapper/crypto
	reboot

If everything went well the system is ready to use. In my case, I added a swap area and have a separated home partition which needs to be added to `/etc/fstab`:

	...
	/dev/vg0/swap	swap	swap	defaults	0 0
	/dev/vg0/home	/home	ext4	defaults	0 0
	...

And to activate swap at boot

	rc-update add swap boot

To set up wifi automatically

	apk add wpa_supplicant wireless-tools
	wpa_passphrase "name" "password" > /etc/wpa_supplicant/wpa_supplicant.conf

The `wpa_supplicant` program manages the wpa and wpa2 authentication and `wireless-tools` has important programs such as `udhcpc` to dynamically get an ip address from a access point.

Look for your wireless interface name by issuing the `ifconfig -a` or `ip link` command. Assuming it's `wlan0`, set the interface behaviour in `/etc/network/interfaces` by adding

	auto wlan0
	iface wlan0 inet dhcp

This tells the kernel to automatically activate the interface and dynamically get an address from an access point.

To connect with a network after boot without manual intervention, the `wpa_supplicant` package installs an init script which can be run by openRC. To activate it, use

	rc-update add wpa_supplicant boot

