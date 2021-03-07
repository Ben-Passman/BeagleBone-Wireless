# BeagleBone Wireless Development Setup
This repo contains a working set of binaries that allow booting via network into a (very) minimal file system, and some associated configuration files for anyone wanting to build their own distribution from scratch. The board will load kernel and device tree binaries via TFTP, and mount a rudimentary file system via NFS. This is meant as a basic setup to allow for further development, so will get a working console running but very little else. There is currently no support for any of the board peripherals (i.e. the WiFi/Bluetooth and HDMI devices).

The kernel and device tree binary are loaded into DDR memory via TFTP, while the root file system is located on the host PC and mounted via NFS.

The host PC in this case is running Debian 10, however the basic setup principles should carry over to other distros.

## Host PC Setup
 ### TFTP Server
- install *xinetd*, *tftp* and *tftpd* packages
 - xinetd is a server to handle network requests
 - tftpd/tftp are the TFTP file server/interface
- create/edit the config file for the xinetd server `sudo vi /etc/xinetd.d/tftp` and add the following lines (Note that the config file syntax requires whitespace characters either side of the equal sign, or the configuration file will not work correctly):
```
service tftp
{
	protocol = udp
    port = 69
    socket_type = dgram
    wait = yes
    user = nobody
    server = /usr/sbin/in.tftpd
    server_args = /var/lib/tftpboot -s
    disable = no
}
```
- create download folder, ensuring it matches the one in the configuration file `sudo mkdir -p /var/lib/tftpboot`
 - set all permissions to read write execute `sudo chmod -R 777 /var/lib/tftpboot`
 - set ownership to nobody `sudo chown -R nobody /var/lib/tftpboot`
- start xinetd server `sudo /etc/init.d/xinetd restart`
- check server status with `sudo /etc/init.d/xinetd status` Error messages will show up here if the server has not been configured correctly


 ### NFS Server
- install the *nfs-kernel-server* package
- create download folder `sudo mkdir -p /srv/nfs/beagle`
- edit /etc/exports and add the following line:
```
/srv/nfs/beagle 192.168.27.2(rw, sync, no_root_squash, no_subtree_check)
```
 - make sure the download folder is correct
 - make sure the IP address matches the address chosen for the board
 - details on NFS server parameters can be found [here](https://linux.die.net/man/5/exports).
- update NFS export table`sudo exportfs -arv`
- restart server `sudo service nfs-kernel-server restart`
- check server status with `sudo service nfs-kernel-server status`

### USB Ethernet connection
As the ethernet connection via USB is not a dedicated physical connection, it will only appear on the host PC when the board tries to connect. The host PC network manager must be set up to recognise the board when it connects, which can be done from the command line with the following:

`nmcli con add type ethernet ifname enxf8dc7a000002 ip4 192.168.27.1/24`
- make sure the MAC address matches the address chosen for the board
- make sure the IP address matches the address chosen for the host PC

### File Setup
- Copy *am335x-boneblack-wireless.dtb* and *uImage* into the NFS folder (*/srv/nfs/beagle*)
- Extract *rootfs.tar.gz* into the TFTP folder (*/var/lib/tftpboot*)

## BeagleBone Setup
- Partition an SD card into two:
 - ~1Gb formatted as FAT16, labelled as "BOOT" and with the boot flag set
 - The remaining space formatted as ext3, labelled as "ROOTFS"
- Copy *MLO*, *u-boot.img* and *uEnv.txt* to the BOOT partition of the SD card

## Boot Process
Insert the SD card and boot the BeagleBone by holding down the S2 button when connecting the USB cable. The board should boot into the remote file system using TFTP and NFS. Assuming a serial cable is connected and running via minicom or similar, the remote file system should be accessible via terminal.

