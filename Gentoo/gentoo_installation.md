# Gentoo Installation:   
**On installation live:**   
Config network and enable ssh:   
```bash
net-setup wlp2s0
```

```bash
passwd
/etc/init.d/sshd start
```

**On remote device:**   
My personal autopartition things:   
```bash
echo -e "p\ng\nn\n1\n\n+256M\nt\n1\nn\n2\n\n+16G\nt\n2\n19\nn\n3\n\n\nw" | fdisk /dev/nvme0n1
mkfs.vfat -F 32 /dev/nvme0n1p1
mkfs.xfs /dev/nvme0n1p3
mkswap /dev/nvme0n1p2
swapon /dev/nvme0n1p2
mount /dev/nvme0n1p3 /mnt/gentoo
cd /mnt/gentoo
```

Get the Stage3 tarball and extract:
```bash
wget <url>
```

```bash
tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
```

Flags:
```bash
nano -w /mnt/gentoo/etc/portage/make.conf
```

```bash
# make.conf
COMMON_FLAGS="-march=native -O2 -pipe"

MAKEOPTS="-j4"

ACCEPT_LICENSE="*"
USE="systemd X \
-elogind"
```

Select mirrors:
```bash
mirrorselect -i -o >> /mnt/gentoo/etc/portage/make.conf
```

```bash
mkdir --parents /mnt/gentoo/etc/portage/repos.conf
cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
```

Mounts and `chroot`:
```bash
mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
mount --make-rslave /mnt/gentoo/dev
mount --bind /run /mnt/gentoo/run
mount --make-slave /mnt/gentoo/run
```

```bash
chroot /mnt/gentoo /bin/bash
```

```bash
source /etc/profile && export PS1="(chroot) ${PS1}"
```

Mount `/boot` and sync stuffz:
```bash
mount /dev/nvme0n1p1 /boot
emerge-webrsync
```

```bash
emerge --sync &&
eselect profile list
```

Set profile and update:
```bash
eselect profile set <NUMBER> &&
emerge --ask --verbose --update --deep --newuse @world
```
**Waiting...**

CPU Flags:
```bash
emerge --ask app-portage/cpuid2cpuflags
```

```bash
echo "*/* $(cpuid2cpuflags)" > /etc/portage/package.use/00cpu-flags
ln -sf ../usr/share/zoneinfo/Europe/Helsinki /etc/localtime
```

Edit and generate locales:
```bash
nano -w /etc/locale.gen
```

```bash
#local.gen
en_US ISO-8859-1
en_US.UTF-8 UTF-8
fi_FI ISO-8859-1
fi_FI.UTF-8 UTF-8
```

```bash
locale-gen
eselect locale list
```

Set locale and update environment:
```bash
eselect locale set <NUMBER> && env-update
```

```bash
source /etc/profile && export PS1="(chroot) ${PS1}"
```

Kernel stuff:
```bash
emerge --ask sys-kernel/linux-firmware
```

```bash
ln -sf /proc/self/mounts /etc/mtab
emerge --ask sys-kernel/gentoo-sources
```

```bash
eselect kernel set 1
ls -l /usr/src/linux
```

```bash
emerge --ask sys-kernel/genkernel
```

```bash
nano -w /etc/fstab
```

```bash
#fstab
/dev/nvme0n1p1  /boot vfat  defaults  0 2
```

Compile kernel:
```bash
genkernel --mountboot --install all
```
**Waiting...**

Check that stuffz are ok before proceeding:
```bash
ls /boot/vmlinu* /boot/initramfs*
blkid
```

Update fstab:
```bash
nano -w /etc/fstab
```

```bash
#fstab
/dev/nvme0n1p1  /boot vfat  defaults  0 2
/dev/nvme0n1p2  none  swap  sw        0 0
/dev/nvme0n1p3  /     xfs  noatime   0 1
```

Add hostname to `hosts` :
```bash
nano /etc/hosts
```

Systemd and NetworkManager things:
```bash
systemd-machine-id-setup
emerge --ask net-misc/networkmanager
```
**Waiting...**

```bash
systemctl enable NetworkManager
```

Set root password:
```bash
passwd
```

Config systemd:
```bash
systemd-firstboot --prompt --setup-machine-id
systemctl preset-all --preset-mode=enable-only
```

Install disk tools (as an example, what i use and need the most):
```bash
emerge --ask sys-fs/e2fsprogs sys-fs/dosfstools sys-fs/xfsprogs
```

Bootloader and Grub stuffz:
```bash
echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf
emerge --ask sys-boot/grub
```

Install Grub:
```bash
grub-install --target=x86_64-efi --efi-directory=/boot --removable
grub-mkconfig -o /boot/grub/grub.cfg
```

Fix OS-Prober
```bash
emerge --ask sys-boot/os-prober
```

Needed to merge/update Flags:
```bash
dispatch-conf
# u = update
``` 

```bash
emerge --ask sys-boot/os-prober
```

```bash
nano -w /etc/default/grub
```

```bash
# grub
GRUB_DISABLE_OS_PROBER=false
```

Update Grub:
```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

Exit chroot environment, unmount stuffz and reboot:
```bash
exit
```

```bash
cd && umount -l /mnt/gentoo/dev{/shm,/pts,} && umount -R /mnt/gentoo
```

```bash
reboot
```

**End of installation, then to configuring stuffz!**

### First configures:
Wifi, `nmtui` is my fave for quick configure with NetworkManager:
```bash
nmtui
```

Login as root.
Install sudo, add user with groups and stuffz:
```bash
emerge --ask app-admin/sudo
```

```bash
useradd -m -G users,wheel,audio -s /bin/bash <USERNAME> && passwd <USERNAME>
```

```bash
visudo
# Give %wheel, sudo permissions
```

**Exit and login with your new user and move to more configurations.**
