# Creating Logical Volume over two disks.
*As playing with Centos 9 Stream, i got interested in LVM, which was pretty new thing for me.*

I did this in the safe place; Debian 11 Virtual Machine, with 2 extra, empty drives connected.     

### Setup:   
```bash
~$ uname -rsv
Linux 5.10.0-20-amd64 #1 SMP Debian 5.10.158-2 (2022-12-13)
~$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   80G  0 disk 
├─sda1   8:1    0 71,2G  0 part /
└─sda2   8:2    0  8,8G  0 part [SWAP]
sdb      8:16   0   10G  0 disk 
sdc      8:32   0   15G  0 disk 
sr0     11:0    1 1024M  0 rom 
```
`sdb` and `sdc` disks are to be combined as a single Logical Volume.  

### Sources i used for this thing:  
https://www.linuxtechi.com/how-to-create-lvm-partition-in-linux/   
https://askubuntu.com/questions/219881/how-can-i-create-one-logical-volume-over-two-disks-using-lvm   
https://blog.khmersite.net/2019/10/how-to-label-xfs-filesystem/   

### How its done

Started by installing needed things, or actually i had installed them before, but im not sure if Debian ships with `lvm2`.    
`xfsprogs` i checked, because i wanted to use XFS:
```bash
~$ sudo apt-get install lvm2 xfsprogs
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
lvm2 is already the newest version (2.03.11-2.1).
xfsprogs is already the newest version (5.10.0-4).
0 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
``` 

**First, i created the Physical Volumes:**
```bash
~$ sudo pvcreate /dev/sdb
WARNING: dos signature detected on /dev/sdb at offset 510. Wipe it? [y/n]: y
  Wiping dos signature on /dev/sdb.
  Physical volume "/dev/sdb" successfully created.
~$ sudo pvcreate /dev/sdc
WARNING: dos signature detected on /dev/sdc at offset 510. Wipe it? [y/n]: y
  Wiping dos signature on /dev/sdc.
  Physical volume "/dev/sdc" successfully created.
```
Then to verify the outcome:
```bash
~$ sudo lvmdiskscan
  /dev/sda1 [     <71,21 GiB] 
  /dev/sda2 [      <8,79 GiB] 
  /dev/sdb  [      10,00 GiB] LVM physical volume
  /dev/sdc  [      15,00 GiB] LVM physical volume
  0 disks
  2 partitions
  2 LVM physical volume whole disks
  0 LVM physical volumes
```
All good!   

**Next step was to create the Volume Group, and assign the needed disks to it:**   
(Just as an example, `volgrp01` is defined as the Volume Group name.)
```bash
~$ sudo vgcreate volgrp01 /dev/sdb /dev/sdc
  Volume group "volgrp01" successfully created
```
Now that it was created, i again, checked the outcome:   
```bash
~$ sudo vgdisplay volgrp01
  --- Volume group ---
  VG Name               volgrp01
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               24,99 GiB
  PE Size               4,00 MiB
  Total PE              6398
  Alloc PE / Size       0 / 0   
  Free  PE / Size       6398 / 24,99 GiB
  VG UUID               GbnIYa-vt4C-ehw6-BFKb-UoDH-Rmj3-hX4IRy
```
Everything looking good so far.   

**Then i created the actual Logical Volume, that will include the previously created Logical Volume Group. Size is defined of what is the maximum.**   
(Again just as an example, i used `lv01` as the name for the Volume)   
```bash 
~$ sudo lvcreate -L 24,99G -n lv01 volgrp01
  Rounding up size to full physical extent 24,99 GiB
  Logical volume "lv01" created.
```
Now when it was created, it needed to be formatted.   

**I wanted to format it as a XFS, so:**   
```bash
~$ sudo mkfs.xfs /dev/volgrp01/lv01
meta-data=/dev/volgrp01/lv01     isize=512    agcount=4, agsize=1637888 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=0
data     =                       bsize=4096   blocks=6551552, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=3199, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```
Done deal!   
As i like to label my drives and stuff, to keep them organized and easily configurable, i labeled the newly created Logical Volume as `New_Volume`, using `xfs_admin`:     
```bash
~$ sudo xfs_admin -L New_Volume /dev/volgrp01/lv01
writing all SBs
new label = "New_Volume"
```

**Then created the mount point:**   
```bash
~$ sudo mkdir /media/$USER/New_Volume
```
Now that its ready to be mounted, the created folder is owned by `root` , i wanted of course to be able to mount my new volume as r/w, so i changed the owner!   
```bash
~$ sudo chown $USER:$USER /media/$USER/New_Volume/
```

**And then to mount, and testings for R/W:**   
```bash
~$ sudo mount /dev/volgrp01/lv01 /media/$USER/New_Volume
~$ touch /media/$USER/New_Volume/Hello_New_Volume
~$ ls -la /media/$USER/New_Volume/
total 4
drwxr-xr-x  2 sicki sicki   30 15. 2. 11:47 .
drwxr-x---+ 6 root  root  4096 15. 2. 11:40 ..
-rw-r--r--  1 sicki sicki    0 15. 2. 11:47 Hello_New_Volume
~$ rm /media/$USER/New_Volume/Hello_New_Volume 
~$ ls -la /media/$USER/New_Volume/
total 4
drwxr-xr-x  2 sicki sicki    6 15. 2. 11:48 .
drwxr-x---+ 6 root  root  4096 15. 2. 11:40 ..
```
All good!   

**Final verification:**  
```bash
~$ lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0   80G  0 disk 
├─sda1            8:1    0 71,2G  0 part /
└─sda2            8:2    0  8,8G  0 part [SWAP]
sdb               8:16   0   10G  0 disk 
└─volgrp01-lv01 254:0    0   25G  0 lvm  /media/sicki/New_Volume
sdc               8:32   0   15G  0 disk 
└─volgrp01-lv01 254:0    0   25G  0 lvm  /media/sicki/New_Volume
sr0              11:0    1 1024M  0 rom  
```
**Wuhuu! Done and done!**