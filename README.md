# Debian-on-NAS326
HOW TO install Debian on Zyxel NAS326

This guide is TL;DR ("short way") of instructions available at https://forum.doozan.com/read.php?2,88619 written by user bodhi. All credit for hacking and installing Debian on this box should go to him. It is advised to follow above link for further details or troubleshooting. 

Prerequisitions:
- Zyxel NAS326
- PC/laptop with Debian installed
- pendrive (note: not every flash drive will work, if your system won't boot try another one)

NOTE: Data stored on internal disks reamain unaffected during the process, so there is a little need to backup them. If something fails, just unplug front usb port, power off machine for few seconds and boot again. Failsafe system image from NAND memory will boot and all settings will be reset.

1. Prepare pendrive with gparted. Create one partition on whole device and format it to ext3, label it as rootfs. Mount.

2. Install package u-boot-tools on your Debian box.

3. Download archive https://www.dropbox.com/s/nkvtlkd9le4ujen/Debian-4.9.0-mvebu-tld-12-rootfs-bodhi.tar.bz2

4. Untar it as user root on prepared pendrive with tar -xjf 

5. Add following line to fstab on pendrive
LABEL=rootfs	/	ext3	noatime,errors=remount-ro	0	1

6. Umoumnt pendrive and insert into fron usb 2.0 port on NAS326

7. Connect via ssh to NAS326 with your admin credentials (you might need to enable ssh service on NAS326: Control panel - Network - Terminal)

8. Store output of command fw_printenv. You can use internal disks mounted somewhere uder /i-data or pendrive (if already inserted) somewhere under /e-data, i.e.: fw_printenv > /i-data/0123456789ABCDEF/fw_printenv-original.txt

9. Apply new settings:
fw_setenv curr_bootfrom 1
fw_setenv next_bootfrom 1
fw_setenv load_dtb_addr 0x1000000
fw_setenv load_initrd_addr 0x2900000
fw_setenv load_image_addr 0x02000000
fw_setenv usb_init 'mw.l f1018100 20420000; mw.l f1018140 003E8800; sleep 3; usb start'
fw_setenv set_bootargs_stock 'setenv bootargs "console=ttyS0,115200 ubi.mtd=4,2048 rootfstype=ubifs root=ubi0:rootfs1 rw rootdelay=2"'
fw_setenv bootcmd_stock_1 'run set_bootargs_stock; echo Booting from NAND kernel 1 ...; nand read 0x2000000 0x00E00000 0xF00000 && bootz 0x2000000'
fw_setenv bootcmd_stock_2 'run set_bootargs_stock; echo Booting from NAND kernel 2 ...; nand read 0x2000000 0x08700000 0xF00000 && bootz 0x2000000'
fw_setenv usb_set_bootargs 'setenv bootargs "console=ttyS0,115200 root=LABEL=rootfs rootdelay=10 $mtdparts earlyprintk=serial"'
fw_setenv usb_bootcmd 'echo Booting from USB ...; setenv fdt_skip_update yes; run usb_init; ext2load usb 0:1 $load_image_addr /boot/zImage; ext2load usb 0:1 $load_dtb_addr /boot/dts/armada-380-zyxel-nas326.dtb; ext2load usb 0:1 $load_initrd_addr /boot/uInitrd; run usb_set_bootargs; bootz $load_image_addr $load_initrd_addr $load_dtb_addr'
fw_setenv bootcmd_custom 'if run usb_bootcmd; then; else if run bootcmd_stock_1; then; else run bootcmd_stock_2; reset; fi; fi'
fw_setenv kernel_addr_1 '0x00000000; run bootcmd_custom; '
fw_setenv change_boot_part 1

10. Store output of fw_printenv again, just in case.

11. Reboot NAS326.

12. Your 'new' Debian 8.0 Jessie should emerge on your LAN with hostname 'debian'. You can locate it using your router software or simply nmap -sP 192.168.1.1/24 | grep debian

13. Login via ssh with user root and password root

