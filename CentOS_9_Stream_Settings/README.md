# Settings for my personal CentOS 9 Stream - Server to Workstation setup.

Disable annoying "beeb beeb" -thing:   
```bash
sudo nano /etc/inputrc
# add/change -> set bell-style none
sudo rmmod pcspkr
sudo nano /etc/modprobe.d/blacklist.conf
# add -> blacklist pcspkr
```

Installing xfce-desktop:   
```bash
# First update stuff
sudo dnf upgrade --refresh

# Enable CRB AKA Code Ready Builder
sudo dnf config-manager --set-enabled crb

# Install EPEL AKA Extra Packages for Enterprise Linux
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-next-release-latest-9.noarch.rpm

# Install Xfce
sudo dnf groupinstall Xfce

# Set xfce4 desktop
echo "exec /usr/bin/xfce4-session" >> ~/.xinitrc

# Set GUI as default
sudo systemctl set-default graphical
sudo reboot
```

Themes:   
```bash
sudo dnf install gnome-themes-extra
sudo dnf install gtk2-engines
```

I have package names in file, which i always want for a new system so:   
Install all packages from list in file:   
```bash
for i in `cat <FILENAME>` ; do sudo dnf -y install $i; done
```

Add ntfs mount-support if necessary:   
```bash
sudo dnf install ntfs-3g fuse ntfsprogs
```

Micro for CentOS:   
```bash
curl https://getmic.ro | bash
sudo mv micro /usr/bin
```

Useful Widget/Plugin/Applet things for myself
```bash
sudo dnf install xfce4-netload-plugin xfce4-systemload-plugin
```

Screensaver to customize screen locking etc:   
```bash
sudo dnf install xscreensaver
```

rpm fusion repos for some packages i needed, for example ffmpeg:   
```bash
sudo dnf install --nogpgcheck https://mirrors.rpmfusion.org/free/el/rpmfusion-free-release-$(rpm -E %rhel).noarch.rpm

sudo dnf install --nogpgcheck https://mirrors.rpmfusion.org/nonfree/el/rpmfusion-nonfree-release-$(rpm -E %rhel).noarch.rpm
```

ffmpeg video codecs to play stuff in browser
```bash
sudo dnf install ffmpeg
```

Installing Vagrant:
```bash
sudo dnf install -y dnf-plugins-core
sudo dnf config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo dnf -y install vagrant
```

To update Grub in CentOS
```bash
sudo grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg
```
