# Building from Scratch
These configuration files were used to generate the project binaries. They are based on the default configurations that are provided along with the source packages.

## Build Sources
Bootloader:  [u-boot 2017.05-rc2](ftp://ftp.denx.de/pub/u-boot/)
Kernel:      [linux 4.14.108](https://github.com/beagleboard/linux)
File System: [busybox 1.26.0](https://www.busybox.net/downloads/)
Toolchain:   [gcc linaro 6.5.0](https://releases.linaro.org/components/toolchain/binaries/6.5-2018.12/arm-linux-gnueabihf/)

## MAC Addresses and Static IPs
- Static IPs are used for the BeagleBone/Host PC connection. These addresses are arbitrary (within allowed local IP values), but must be consistent across board and PC setups.
- MAC addresses are used to set up  the ethernet gadget module (g_ether), these are also arbitrary but once again must remain consistent between the board and the host PC.

All these addresses are assigned during the BeagleBone boot process, and are contained in the u-boot *uEnv.txt* file.

## Bootloader (u-boot)
There is a default configuration available for the BeagleBone board (am335x_boneblack_defconfig), which should not require any additional modifications.
The relevant connection settings for the USB ethernet connection can be found under the "Device Drivers -> USB Support" menu of the configuration tool. "USB Gadget Support" should be enabled, and "Enable USB download agent" selected in the sub-menu.

### uEnv.txt Setup
U-boot sets up the board according to the uEnv.txt file, which in this case does the following:
- sets the ethernet device to use the USB gadget
- loads the linux kernel and device tree binary into DDR memory via TFTP
- sets the boot arguments to be passed to the linux kernel
 - sets the console device
 - sets the MAC addresses for the USB gadget
 - sets the NFS server command line parameters
- boots the linux kernel from DDR memory

```
ethact=usb_ether # Use USB ethernet gadget
console=ttyS0,115200n8
ipaddr=192.168.27.2
serverip=192.168.27.1
ethargs=g_ether.dev_addr=f8:dc:7a:00:00:02 g_ether.host_addr=f8:dc:7a:00:00:01
### TFTP load commands ###
loadtftp=echo Booting from network...; tftpboot ${loadaddr} uImage; tftpboot ${fdtaddr} am335x-boneblack-wireless.dtb
### NFS settings ###
rootpath=/srv/nfs/bbb
nfsopts=nfsvers=3,tcp,nolock,wsize=1024,rsize=1024
nfsdev=usb0
### Command to set Linux kernel boot arguments ###
setbootargs=setenv bootargs console=${console} ${ethargs} root=/dev/nfs rw nfsroot=${serverip}:${rootpath},${nfsopts} rootwait rootdelay=5 ip=${ipaddr}:::::${nfsdev}
### Boot ###
uenvcmd=setenv autoload no; run loadtftp; run setbootargs; bootm ${loadaddr} - ${fdtaddr}

```
#### Notes:
Ensure there is an empty line at the end of the uEnv.txt file
Further information on the NFS server boot arguments can be found [here](https://www.kernel.org/doc/html/latest/admin-guide/nfs/nfsroot.html)

## Kernel
As the BeagleBone wireless board lacks a physical ethernet connection, ethernet over USB is required to allow the board to mount a remote file system. As kernel modules will not be available until the file system has been mounted, the USB gadget (g_ether) needs to be built-in to the kernel for this functionality to be available.

The default configuration file (bb.org_defconfig) provides a good starting point. To configure the USB gadget as a built-in module, the following settings should be checked:
- Device Drivers -> USB support -> USB Gadget Support should be set to built-in
- "USB Gadget precomposed configurations" (under USB Gadget Support) should be set to "Ethernet Gadget", and should also be built-in.