# rootOnUSB
Set rootfs to be on a USB drive

Original article on JetsonHacks: https://wp.me/p7ZgI9-317

<em><b>WARNING: </b>This is a low level system change. You may have issues which are not easily solved. You should do this working on a freshly flashed micro SD card, and certainly do not attempt this with valuable data on the card itself. Assume that if this does not work, you may have to flash the micro SD card again. A serial debug console is useful if things go wrong. </em>

Note1: Even if you can boot the jetpack on USB installed drive, you must have the micro sd card because in first step boot process will be start on micro sd card then will be changed to USB hard drive.

The scripts in this repository will setup a NVIDIA Jetson Nano Developer Kit to set the rootfs to a USB drive. This involves four steps. <em><b>NOTE: </b>This procedure is significantly different than the previous release of this repository. This version does not require the kernel to be recompiled, saving about 40 minutes of install time.</em>
## Step 1
Build the initramfs with USB support, so that USB is available early in the boot process. A convenience script named addUSBToInitramfs.sh provides this functionality.

```
$ ./addUSBToInitramfs.sh
```

## Step 2
The second step does not have representation here. The user must prepare a USB drive (preferably USB 3.0, SSD, HDD, or SATA->USB) by formatting the disk as ext4 with a partition. It is easier if you only plug in one USB drive during this procedure. When finished, the disk should show as /dev/sda1 or similar. Note: Make sure that the partition is ext4, as NTSF will appear to copy correctly but cause issues later on. Typically it is easiest to set the volume label for later use during this process.

## Step 3
Copy the application area of the micro SD card to the USB drive. copyRootToUSB.sh copies the contents of the entire system micro SD card to the USB drive. Naturally, the USB drive storage should be larger than the micro SD card. Note: Make sure that the USB drive is mounted before running the script. In order to copyRootToUSB:

```
usage: ./copyRootToUSB.sh [OPTIONS]

  -d | --directory     Directory path to parent of kernel

  -v | --volume_label  Label of Volume to lookup

  -p | --path          Device Path to USB drive (e.g. /dev/sda1)

  -h | --help  This message
  ```

## Step 4
Modify the /boot/extlinux/extlinux.conf file. An entry should be added to point to the new rootfs (typically this is /dev/sda1). There is a sample configuration file: sample-extlinux.conf

You should make a backup of the original extlinux.conf file. Also, when you edit the file you should make a backup of the original configuration and relabel the backup. This will allow you to access an alternate boot method from the serial console in case something goes sideways.

Then you should changed the INITRD line to:

  ```
INITRD /boot/initrd-xusb.img
```

So that the system uses the initramfs that we built that includes the USB firmware. Then set the root to the USB drive.

Here are some examples. You can set the drive by the UUID of the disk drive, the volume label of the drive, or the device path:

```
APPEND ${cbootargs} root=UUID=0e437280-bea0-42a2-967f-a240dd3075eb rootwait rootfstype=ext4
APPEND ${cbootargs} root=LABEL=JetsonNanoSSD500 rootwait rootfstype=ext4
APPEND ${cbootargs} root=/dev/sda1 rootwait rootfstype=ext4
  ```

The first entry is most specific, the last most generic. Note that you are not guaranteed that a USB device is enumerated in a certain order and will always have the same device path. That is, if you leave another USB drive plugged in along with your root disk when you boot the Jetson, the root disk may have a different path than originally, such as /dev/sdb1.  

Also, there is a convenience file: diskUUID.sh which will determine the UUID of a given device. This is useful for example to determine the UUID of the USB drive. Note: If the UUID returned is not similar in length to the above example, then it is likely that the device is not formatted as ext4.

```
$ ./diskUUID.sh
```

While this defaults to sda1 (/dev/sda1), you can also determine other drive UUIDs. The /dev/ is assumed, use the -d flag. For example:

```
$ ./diskUUID.sh -d sdb1
```
You may find this information useful for setting up the extlinux.conf file

<h2>Release Notes</h2>
<h3>November, 2019</h3>

* Jetson Nano
* L4T 32.2.3
* Linux kernel 4.9-140
* No changes, release only to match L4T

<h3>September, 2019</h3>

* Jetson Nano
* L4T 32.2.1 (JetPack 4.2.2)
* Linux kernel 4.9.140
* Change from recompiling kernel to include the tegra-xusb driver, to adding the tegra-xusb to initramfs. This allows access to the usb driver early on in the boot process.


<h3>July, 2019</h3>

* Initial Release
* Jetson Nano
* L4T 32.2 (JetPack 4.2.1)
* Linux kernel 4.9.140

<h3>April, 2019</h3>

* Initial Release
* Jetson Nano
* L4T 32.1.0 (JetPack 4.2)
* Linux kernel 4.9.140
