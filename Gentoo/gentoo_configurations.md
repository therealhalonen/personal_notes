# Gentoo configurations   

SSH after installing Gentoo:
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
**Disable light-dm background change for user:**
```bash
echo "user-background=false" | sudo tee -a /etc/lightdm/lightdm-gtk-greeter.conf 
```

**Get `micro` and `xclip`**
```bash
    curl https://getmic.ro | bash
    sudo mv micro /usr/bin
    sudo emerge -a x11-misc/xclip
```

**Fix Pulseaudio:**
```bash
sudo emerge -a media-sound/pavucontrol
sudo micro /etc/xdg/autostart/pulseaudio.desktop
# Change start to:
pulseaudio --start
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

**Install Virtualbox 6.1.42 ; as im not comfortable yet with 7.X:**   
https://wiki.gentoo.org/wiki/VirtualBox   

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
gpasswd -a <USERNAME> vboxusers
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

**Better lockscreen (/w guru repository):**      
First install eselect-repository, enable guru and sync:
```bash
sudo emerge -avt app-eselect/eselect-repository
sudo eselect repository enable guru
sudo emerge --sync
```

Mask whole guru repo:   
```bash
sudo tee -a /etc/portage/package.mask/guru <<< "*/*::guru"
```
To unmask single package to install from guru.   
```bash
sudo mkdir -p /etc/portage/package.unmask
sudo tee -a /etc/portage/package.unmask/package <<< ">=category/package»::guru"
```

To install `betterlockscreen`:   
https://github.com/betterlockscreen/betterlockscreen   

Unmask package and dependency:
```bash
echo " x11-misc/betterlockscreen **" | sudo tee /etc/portage/package.accept_keywords/betterlockscreen
echo " x11-misc/i3lock-color **" | sudo tee /etc/portage/package.accept_keywords/i3lock-color
echo ">=x11-misc/betterlockscreen-9999::guru" | sudo tee /etc/portage/package.unmask/betterlockscreen
echo ">=x11-misc/i3lock-color-9999::guru" | sudo tee /etc/portage/package.unmask/i3lock-color
```
Install:
```bash
sudo emerge -a x11-misc/betterlockscreen
```
With `xfce4` add lockscreen command to `xflock4`:   
```bash
sudoedit /usr/bin/xflock4
```
```bash
#xflock4
for lock_cmd in \
    betterlockscreen \ # Add this line
```

Force uninstall `xfce4-screensaver` and mask it:   
```bash
sudo emerge --unmerge xfce-extra/xfce4-screensaver
echo "xfce-extra/xfce4-screensaver" | sudo tee /etc/portage/package.mask/xfce4-screensaver
```
Now, updating packages, will install `xscreensaver`, as `xfce4`, its okay, install it and disable everything from `xscreensaver` settings afterward. Also disable autolaunch for it

Fix betterlockscreen daemon by replacing exec= `simple` with `forking`
```bash
sudoedit /lib/systemd/system/betterlockscreen\@.service
```

Pro tip, to myself:   
If installing Gentoo to another PC, and wanting to get the list of installed packages from another Gentoo installation:
```bash
# Create a file with packages currently @world
cat /var/lib/portage/world > to_be_installed
# Re-arrange the package list and merge all
sudo emerge -a $(tr "\n" " " <installed)
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

**Fix for `dunst` if installed; remove other notification daemons:**
```bash
emerge -a --depclean x11-misc/notification-daemon
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

# Unmask package
echo " <package_name> **" >> /etc/portage/package.accept_keywords/<package_name>

# Update Flags:
sudo dispatch-conf

# Mask a repo, guru for example: 
tee -a /etc/portage/package.mask/guru <<< "*/*::guru"

# To unmask single package from specific repo:
tee -a /etc/portage/package.unmask/package <<< "category/package»::repo"
```
