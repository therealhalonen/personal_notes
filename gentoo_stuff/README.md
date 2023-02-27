# Gentoo
*My personal notes and stuff, about installing Gentoo.   
Not meant to be used as any guide or anything, as may contain errors and not recommended fixes.   
Wrote these, to make future installations quicker and to remember what i did etc...*

**Special thanks, as most, if not all the commands and stuffs in installation part are from my friend's guide:**  
https://github.com/Veliquu/Personal-linux

Some other sources used here, and should be checked before, during and after Gentoo installation:   
https://wiki.gentoo.org/wiki/Main_Page   
https://wiki.gentoo.org/wiki/Handbook:AMD64/Full/Installation   
https://wiki.gentoo.org/wiki/Systemd   
https://wiki.gentoo.org/wiki/Xfce/Guide   

### Installation:   
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

Get the Stage3 tarball:
```bash
wget <URL> --xattrs-include='*.*' --numeric-owner
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

**Exit and login with your new user.**

Optional SSH:
```bash
sudo systemctl enable sshd && sudo systemctl startsshd
```
Login to ssh with your user from remote machine if preferred.

### Install and configure `xfce4` DE

Install xfce4-meta:   
```bash
sudo emerge --ask xfce-base/xfce4-meta
```

Needed to merge/update Flags:
```bash
sudo dispatch-conf
```

```bash
sudo emerge --ask xfce-base/xfce4-meta
```
**Waiting...**

This was from the guide, i dont think it was needed, but will leave it here as a note:
```bash
for x in cdrom cdrw usb ; do sudo gpasswd -a <USERNAME> $x ; done
```

Personally needed `xfce4` plugins and stuffz:
```bash
sudo emerge --ask xfce-extra/xfce4-pulseaudio-plugin xfce-extra/xfce4-taskmanager x11-themes/xfwm4-themes app-editors/gedit xfce-base/xfce4-power-manager x11-terms/xfce4-terminal xfce-base/thunar xfce-extra/xfce4-battery-plugin xfce-extra/xfce4-mount-plugin xfce-extra/xfce4-sensors-plugin xfce-extra/xfce4-netload-plugin xfce-extra/xfce4-systemload-plugin x11-misc/lightdm xfce4-screenshooter gnome-extra/nm-applet
```

Append to flags, if wanting to probe sensors and show them in plugin:
```bash
>=xfce-extra/xfce4-sensors-plugin-1.4.4 lm-sensors
```

Pretty straight forward to enable `xfce4` and `light-dm`
```bash
echo "exec startxfce4" > ~/.xinitrc
```

```bash
echo XSESSION=\"Xfce4\" | sudo tee /etc/env.d/90xsession
sudo env-update && source /etc/profile
```

```bash
sudo systemctl enable lightdm
sudo systemctl start lightdm
```

Reboot.

### Tweaks and notes:
Disable light-dm background change for user:
```bash
echo "user-background=false" | sudo tee -a /etc/lightdm/lightdm-gtk-greeter.conf 
```

Install Firefox-esr without compiling it from source:
```bash
sudo emerge -a www-client/firefox-bin:esr
```

Unmas and install `light-locker`, as i want it:
```bash
echo " x11-misc/light-locker **" >> /etc/portage/package.accept_keywords/light-locker
emerge -a x11-misc/light-locker
```

Get `micro` and `clip`
```bash
    curl https://getmic.ro | bash
    sudo mv micro /usr/bin
    sudo emerge -a x11-misc/xclip
```

Fix Pulseaudio:
```bash
sudo emerge -a media-sound/pavucontrol
sudo micro /etc/xdg/autostart/pulseaudio.desktop
# Change start to:
pulseaudio --start
```

### Troubleshoot and that kind of stuff:

**If suspend freezes stuffz, with laptop lid closed/opened**   
```
    sudo nano /etc/systemd/logind.conf
```
Add the following line: (If it already exists, edit or uncomment):   
```
    HandleLidSwitch=ignore
```

**Keymap stuff, if need to edit:**
```
sudo nano /etc/vconsole.conf
```

**Chroot from Live, if problems:**
```
# Create mounting directories  
mkdir /mnt/gentoo  
cd /mnt/gentoo  

# Activate swap  
swapon /dev/nvme0n1p2

# Mount partitions  
mount /dev/nvme0n1p3 /mnt/gentoo  
mount /dev/nvme0n1p1 /mnt/gentoo/boot  
mount -t proc none /mnt/gentoo/proc  
mount -o bind /dev /mnt/gentoo/dev  
   
# Copy network information so we can use the net inside chroot  
cp -L /etc/resolv.conf /mnt/gentoo/etc/resolv.conf  
   
# Chroot into system  
chroot /mnt/gentoo /bin/bash  
env-update  
source /etc/profile  
export PS1="(chroot) $PS1"  
   
# Do stuffz!

# After youre done exit chroot:  
exit  
   
# Unmount  
umount /mnt/gentoo/boot /mnt/gentoo/dev /mnt/gentoo/proc /mnt/gentoo  
   
# Deactivate swap  
swapoff /dev/nvme0n1p2  
   
# Return to user prompt  
exit
```

**ZSH, with autosuggestions, completions and syntax-highlighting (Kali style):**
```bash
emerge --ask app-shells/zsh app-shells/zsh-completions app-shells/zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-autosuggestions ~/.zsh/zsh-autosuggestions
```
Add to .zshrc:   
```bash
source /usr/share/zsh/site-functions/zsh-syntax-highlighting.zsh
source ~/.zsh/zsh-autosuggestions/zsh-autosuggestions.zsh
```

**Install Virtualbox 6.1.42 ; as im not comfortable yet with 7.X**
```bash
sudo emerge --ask =app-emulation/virtualbox-6.1.42
sudo emerge --ask =app-emulation/virtualbox-additions-6.1.42
```
Mask version to not update later:    
```bash
echo ">app-emulation/virtualbox-6.1.42" | sudo tee /etc/portage/package.mask/virtualbox-6.1.42
echo ">app-emulation/virtualbox-additions-6.1.42" | sudo tee /etc/portage/package.mask/virtualbox-additions-6.1.42
```

Add to group and stuffz:
```bash
gpasswd -a <PASSWORD> vboxusers
```

Virtualbox kernel module things:
```bash
emerge --ask --oneshot @module-rebuild
modprobe vboxdrv
modprobe vboxnetadp
modprobe vboxnetflt 
```

```bash
systemctl start systemd-modules-load
```

**Install Vagrant**
```bash
#Not sure if needed: dev-lang/ruby
```

https://developer.hashicorp.com/vagrant/docs/installation/source

If errors when running Vagrant from /usr/local/bin:
```bash
hash -r
```
Append to .bashrc to stop Vagrant to give warnings about "unofficial installation"
```bash
export VAGRANT_I_KNOW_WHAT_IM_DOING_PLEASE_BE_QUIET=true
```

**Portage and system notes and things:**
```bash
# Update repos
emerge --sync

# Upgrade packages
emerge -avuDN @world

# Search packages
emerge --search <package>

# Useful Portage log for postinst message:
sudo cat /var/log/portage/elog/summary.log

# List all isntalled packages:
cat /var/lib/portage/world
```

**My current installed packages (27.02.2023):**
```bash
app-admin/salt
app-admin/sudo
app-arch/p7zip
app-arch/unzip
app-editors/gedit
app-emulation/virtualbox
app-emulation/virtualbox-additions
app-misc/neofetch
app-portage/cpuid2cpuflags
app-shells/zsh
app-shells/zsh-completions
app-shells/zsh-syntax-highlighting
app-text/tree
dev-lang/ruby
dev-vcs/git
gnome-extra/nm-applet
media-gfx/gimp
media-sound/pavucontrol
media-video/simplescreenrecorder
media-video/vlc
net-im/discord
net-misc/networkmanager
sys-boot/grub
sys-boot/os-prober
sys-fs/dosfstools
sys-fs/xfsprogs
sys-kernel/genkernel
sys-kernel/gentoo-sources
sys-kernel/linux-firmware
www-client/firefox-bin:esr
x11-misc/light-locker
x11-misc/lightdm
x11-misc/xclip
x11-terms/xfce4-terminal
x11-themes/gnome-colors-common
x11-themes/gnome-colors-themes
x11-themes/gtk-engines
x11-themes/gtk-engines-adwaita
x11-themes/xfwm4-themes
xfce-base/thunar
xfce-base/xfce4-meta
xfce-base/xfce4-power-manager
xfce-extra/xfce4-battery-plugin
xfce-extra/xfce4-mount-plugin
xfce-extra/xfce4-netload-plugin
xfce-extra/xfce4-pulseaudio-plugin
xfce-extra/xfce4-screenshooter
xfce-extra/xfce4-sensors-plugin
xfce-extra/xfce4-systemload-plugin
xfce-extra/xfce4-taskmanager
```
