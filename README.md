```bash
parted -a optimal /dev/sda

#mklabel gpt

unit mib
mkpart primary 1 3
name 1 grub
set 1 bios_grub on

mkpart primary 3 131
name 2 boot
set 2 boot on

mkpart primary 131 643
name 3 swap

mkpart primary 643 -1
name 4 rootfs

#mkfs.ext4

mount /dev/sda4 /mnt/gentoo
date
cd /mnt/gentoo
links https://www.gentoo.org/downloads/mirrors/

tar xpvf stage3-*.tar.bz2 --xattrs-include='*.*' --numeric-owner

nano -w /mnt/gentoo/etc/portage/make.conf
	COMMON_FLAGS="-march=native -O2 -pipe"
	MAKEOPTS="-j9"
	ACCEPT_KEYWORDS="~amd64"
	L10N="en-US zh-CN en zh"
	LANGUAS="en_US zh_CN en zh"
	#ACCEPT_LICENSE="*"
	
mirrorselect -i -o >> /mnt/gentoo/etc/portage/make.conf
mkdir --parents /mnt/gentoo/etc/portage/repos.conf

cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf

cp --dereference /etc/resolv.conf /mnt/gentoo/etc/

mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
mount --make-rslave /mnt/gentoo/dev

chroot /mnt/gentoo /bin/bash
source /etc/profile
export PS1="(chroot) ${PS1}"

mkdir /boot
mount /dev/sda2 /boot

emerge-webrsync
emerge --sync --quiet

eselect profile list
emerge --ask --verbose --update --deep --newuse @world

ls /usr/share/zoneinfo
echo "Asia/Shanghai" > /etc/timezone
emerge --config sys-libs/timezone-data
nano -w /etc/locale.gen
locale-gen
eselect locale list
env-update && source /etc/profile && export PS1="(chroot) $PS1"


emerge --ask sys-kernel/gentoo-sources
ls -l /usr/src/linux
emerge --ask sys-apps/pciutils
emerge --ask sys-kernel/genkernel
nano -w /etc/fstab
blkid
	/dev/sda2   /boot        ext2    defaults,noatime     0 2
	/dev/sda3   none         swap    sw                   0 0
	/dev/sda4   /            ext4    noatime              0 1
  	/dev/cdrom  /mnt/cdrom   auto    noauto,user          0 0

genkernel all
emerge --ask sys-kernel/linux-firmware

nano -w /etc/hostname
nano -w /etc/issue #去掉.\O

passwd

emerge --ask sys-process/cronie
emerge --ask sys-apps/mlocate
emerge --ask net-misc/dhcpcd
emerge -av networkmanager

echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf
emerge --ask --verbose sys-boot/grub:2
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Gentoo
grub-mkconfig -o /boot/grub/grub.cfg
#Could not prepare Boot variable: Read-only file system
mount -o remount,rw /sys/firmware/efi/efivars

exit
cd
umount -l /mnt/gentoo/dev{/shm,/pts,}
umount -R /mnt/gentoo
reboot




useradd -m -G users,wheel,audio -s /bin/bash yume
passwd yume

rm /stage3-*.tar.*

USE="-qt4 -qt5 -kde X gtk gnome systemd"

emerge --ask gnome-base/gnome-light
env-update && source /etc/profile

```

