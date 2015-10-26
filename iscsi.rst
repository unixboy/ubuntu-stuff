HOW TO CONFIGURE ISCSI TARGET AND ISCSI INITIATOR USING UBUNTU 14.04.1

April 6, 2015 admin

This lab will configure iSCSI Target and iSCSI Initiator using Ubuntu 14.0.1.

Topology

This lab will use two machines with static IP addresses:

 – Target.dalaris.local (Target / Server): IP address 10.0.0.10
– Initiator.dalaris.local (Initiator / Client): IP address 10.0.0.11

In this lab setup, the target server has a few hard drives. We will use sda and create a file called /storage/lun1.img. This file has a capacity of 5GB. The initiator connects to the target server and sees the file and turns it into its own disk as sde. We will create an EXT4 partition for disk sde and call it sde1. So to the initiator, it is a disk but to the target, it is actually a file.



Assume that both servers have Ubuntu Linux 14.04.1 installed and configured properly.

Note: Install Ubuntu Linux 14.04.1 on both servers, to list all NIC cards on any server, use command:

#lspci | grep –i eth

We need root access to execute most of the commands, so please elevate the account previlege:

#sudo su

Type the root user password as Pass1234

Configure the Target server

Configure static IP address

auto eth0
iface eth0 inet static
address 10.0.0.100
netmask 255.255.255.0
gateway 10.0.0.1

Configure DNS Servers

#vi /etc/resolv.conf
search dalaris.local
nameserver 8.8.8.8
nameserver 8.8.4.4

Restart service: service networking restart

Test connectivity by pinging google.com, make sure you get a reply.

#ping google.com

Use the following command to install on Target server:

#aptitude install iscsitarget iscsitarget-source iscsitarget-dkms

Note: If the above command fails, use the following commands to update/upgrade the server first.

#sudo apt-get update

#sudo apt-get upgrade

When installation is completed, open file /etc/default/iscsitarget…

#vi /etc/default/iscsitarget

Change the value ISCSITARGET_ENABLE to true (this is to enable iSCSI target):

ISCSITARGET_ENABLE=true

For an iSCSI LUN, we can use any unused logical volumes, image files, hard drives (e.g. /dev/sde), hard drive partitions (e.g. /dev/sde1) or even RAID devices (e.g. /dev/md0) for the storage.

In our setup we are using an image file for our target LUN. The file can be created as follows:

Make a directory

# mkdir /storage

Create an empty file of 5GB whose name is lun1.img, store the file in the /storage directory.

# dd if=/dev/zero of=/storage/lun1.img bs=1024k count=5000

This process will create an image file /storage/lun1.img with volume of 5 GB.

Next, we will adjust the file /etc/ietd.conf…:

#vi /etc/iet/ietd.conf

Comment out every line in the file (by default this is already the case). At the end of the file, add the following lines:

Target iqn.2014-10.local.dalaris:storage.lun1

        IncomingUser chuong secret

        OutgoingUser

        Lun 0 Path=/storage/lun1.img,Type=fileio

        Alias LUN1

Note: The target name must be a unique name, the iSCSI standard defines the iQN (iSCSI Qualified Name) as follows: iqn.yyyy-mm.<reversed domain name>[:identifier];

where yyyy-mm is the date at which the domain is valid; this identifier can be anything you choose.

The IncomingUser line contains a username and a password so that only the initiators (clients) that provide this username and password can log in and use the storage device. If we do not need authentication, don’t specify a username and password in the IncomingUser line. Leave that line as:

IncomingUser

In the Lun line, we must specify the full path to the storage device (e.g. /dev/vg0/storage_lun1, /storage/lun1.img, /dev/sda, etc.). In our case it is /storage/lun1.img.

We need to configure the access permission for iSCSI traget lun. Allow connection to the device iqn.2014-10.local.dalaris:storage.lun1 from 10.0.0.11.

# vi /etc/iet/initiators.allow

iqn.2014-10.local.dalaris:storage.lun1    10.0.0.11

If we would like all clients to connect, leave this line out.

Start the target:

#/etc/init.d/iscsitarget start

or

#service iscsitarget restart

Test port:

#netstat –tulpn | grep 3260

Install the Initiator (server1 at 10.0.0.11)

On server1 (the client), we install the initiator:

aptitude install open-iscsi

Congiure /etc/iscsi/iscsid.conf… :

vi /etc/iscsi/iscsid.conf

change the value of node.startup thành automatic:

[…]
node.startup = automatic
[…]

Restart the initiator:

/etc/init.d/open-iscsi restart

We need to check and see what the target server has:

#iscsiadm -m discovery -t st -p 10.0.0.10

Or enter this command:

#iscsiadm –mode discovery –type sendtarget –portal 10.0.0.10

The result should be: 10.0.0.10:3260,1 iqn.2014-10.local.dalaris:storage.lun1

#iscsiadm -m node

The result should be 10.0.0.10:3260,1 iqn.2014-10.local.dalaris:storage.lun1

 

The target storage iqn.2014-10.local.dalaris:storage.lun1 on 10.0.0.10:3260,1 is stored in the file /etc/iscsi/nodes/iqn.2014-10.local.example:storage.lun1/10.0.0.10,3260,1/default.

If we have setup IncomingUser credentials inside the file /etc/iet/ietd.conf, we must now initialize a username and password for the target. The iscsiadm commands are employed for this purpose:

The first command specifies the authentication method, which is CHAP.

iscsiadm -m node –targetname “iqn.2014-10.local.dalaris:storage.lun1″ –portal “10.0.0.10:3260″ –op=update –name node.session.auth.authmethod –value=CHAP

The second command specifies the authentication username, which is chuong.

iscsiadm -m node –targetname “iqn.2014-10.local.dalaris:storage.lun1″ –portal “10.0.0.10:3260″ –op=update –name node.session.auth.username –value=chuong

The third command specifies the authentication password, which is secret.
iscsiadm -m node –targetname “iqn.2014-10.local.dalaris:storage.lun1″ –portal “10.0.0.10:3260″ –op=update –name node.session.auth.password –value=secret

After specifying the login parameters, we then proceed to login to the target by using the following command:

# iscsiadm -m node –targetname “iqn.2014-10.local.dalaris:storage.lun1″ –portal “10.0.0.10:3260″ –login


The system will show:

Logging in to [iface: default, target: iqn.2014-10.local.dalarus:storage.lun1, portal: 10.0.0.10,3260]
Login to [iface: default, target: iqn.2014-10.local.dalaris:storage.lun1, portal: 10.0.0.10,3260]: successful

 

Note: To restart initiator, use the following command:

#/etc/init.d/open-iscsi restart

or

#service open-iscsi restart

Note: To log out of the target, use the command:

#iscsiadm -m node –targetname “iqn.2014-10.local.dalaris:storage.lun1″ –portal “10.0.0.10:3260″ –logout

The disk is now available without partitions. To check the disk, use the fdisk or dmesg commands:

#fdisk –l

#dmesg | grep sd

We will see a new disk (in this lab it is /dev/sde) – that is our iSCSI disk:

#fdisk -l

Disk /dev/sda: 500.0 GB, 32212254720 bytes
255 heads, 63 sectors/track, 3916 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00016be9

Device Boot Start End Blocks Id System
/dev/sda1 * 1 32 248832 83 Linux
Partition 1 does not end on cylinder boundary.
/dev/sda2 32 3917 31205377 5 Extended
/dev/sda5 32 3917 31205376 8e Linux LVM

Disk /dev/sde: 5.3 GB, 21474836480 bytes
64 heads, 32 sectors/track, 20480 cylinders
Units = cylinders of 2048 * 512 = 1048576 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000

Disk /dev/sde doesn’t contain a valid partition table
root@target:~#

To use the disk, we need to create a partition within the disk:

# fdisk /dev/sde


Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel. Building a new DOS disklabel with disk identifier 0x882944df.
Changes will remain in memory only, until you decide to write them.
After that, of course, the previous content won’t be recoverable.

The number of cylinders for this disk is set to 20480.
There is nothing wrong with that, but this is larger than 1024,
and could in certain setups cause problems with:
1) software that runs at boot time (e.g., old versions of LILO)
2) booting and partitioning software from other OSs
(e.g., DOS FDISK, OS/2 FDISK)
Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)

Command (m for help): <– type m
Command action
a toggle a bootable flag
b edit bsd disklabel
c toggle the dos compatibility flag
d delete a partition
l list known partition types
m print this menu
n add a new partition
o create a new empty DOS partition table
p print the partition table
q quit without saving changes
s create a new empty Sun disklabel
t change a partition’s system id
u change display/entry units
v verify the partition table
w write table to disk and exit
x extra functionality (experts only)

Command (m for help): <– type n
Command action
e extended
p primary partition (1-4)
<– press p
Partition number (1-4): <– press 1
First cylinder (1-20480, default 1): <– press ENTER
Using default value 1
Last cylinder or +size or +sizeM or +sizeK (1-20480, default 20480): <– press ENTER
Using default value 20480

Command (m for help): <– press t
Selected partition 1
Hex code (type L to list codes): <– press L

0 Empty 1e Hidden W95 FAT1 80 Old Minix be Solaris boot
1 FAT12 24 NEC DOS 81 Minix / old Lin bf Solaris
2 XENIX root 39 Plan 9 82 Linux swap / So c1 DRDOS/sec (FAT-
3 XENIX usr 3c PartitionMagic 83 Linux c4 DRDOS/sec (FAT-
…..
1b Hidden W95 FAT3 70 DiskSecure Mult bb Boot Wizard hid ff BBT
1c Hidden W95 FAT3 75 PC/IX
Hex code (type L to list codes): <– choose 83

Command (m for help): <– press w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
target:~#

Now use fdisk again:

#fdisk -l

Disk /dev/sda: 500 GB, 32212254720 bytes
255 heads, 63 sectors/track, 3916 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00016be9

Device Boot Start End Blocks Id System
/dev/sda1 * 1 32 248832 83 Linux
Partition 1 does not end on cylinder boundary.
/dev/sda2 32 3917 31205377 5 Extended
/dev/sda5 32 3917 31205376 8e Linux LVM

Disk /dev/sdb: 5.3 GB, 21474836480 bytes
64 heads, 32 sectors/track, 20480 cylinders
Units = cylinders of 2048 * 512 = 1048576 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x725b9dff

Device Boot Start End Blocks Id System
/dev/sde1 1 20480 20971504 83 Linux
root@target:~#

Now we will prepare the partition /dev/sde1 to be of file system ext4:

#mkfs.ext4 /dev/sde1

Now we can mount the partition to a mount point called /mnt:

#mount /dev/sde1 /mnt

Check the mount points:

#mount


/dev/mapper/server1-root on / type ext4 (rw,errors=remount-ro)
proc on /proc type proc (rw,noexec,nosuid,nodev)
none on /sys type sysfs (rw,noexec,nosuid,nodev)
none on /sys/fs/fuse/connections type fusectl (rw)
none on /sys/kernel/debug type debugfs (rw)
none on /sys/kernel/security type securityfs (rw)
none on /dev type devtmpfs (rw,mode=0755)
none on /dev/pts type devpts (rw,noexec,nosuid,gid=5,mode=0620)
none on /dev/shm type tmpfs (rw,nosuid,nodev)
none on /var/run type tmpfs (rw,nosuid,mode=0755)
none on /var/lock type tmpfs (rw,noexec,nosuid,nodev)
none on /lib/init/rw type tmpfs (rw,nosuid,mode=0755)
none on /var/lib/ureadahead/debugfs type debugfs (rw,relatime)
/dev/sda1 on /boot type ext2 (rw)
/dev/sde1 on /mnt type ext4 (rw)
root@target:~#

Check the disk using the df command:

#df -h 

Filesystem Size Used Avail Use% Mounted on
/dev/mapper/server1-root
18G 838M 16G 5% /
none 243M 180K 242M 1% /dev
none 247M 0 247M 0% /dev/shm
none 247M 36K 247M 1% /var/run
none 247M 0 247M 0% /var/lock
none 247M 0 247M 0% /lib/init/rw
none 18G 838M 16G 5% /var/lib/ureadahead/debugfs
/dev/sda1 228M 17M 199M 8% /boot
/dev/sdb1 20G 172M 19G 1% /mnt
root@target:~#

to unmount the partition, use the following command:

#umount /mnt

To automatically connect to target on startup, we create a directory /storage

#mkdir /storage

add this line to the file /etc/fstab:

#vi /etc/fstab

[…]
/dev/sde1 /storage ext4 defaults,auto,_netdev 0 0

Restart the server system and test the mount points again:

#mount

/dev/mapper/server1-root on / type ext4 (rw,errors=remount-ro)
proc on /proc type proc (rw,noexec,nosuid,nodev)
none on /sys type sysfs (rw,noexec,nosuid,nodev)
none on /sys/fs/fuse/connections type fusectl (rw)
none on /sys/kernel/debug type debugfs (rw)
none on /sys/kernel/security type securityfs (rw)
none on /dev type devtmpfs (rw,mode=0755)
none on /dev/pts type devpts (rw,noexec,nosuid,gid=5,mode=0620)
none on /dev/shm type tmpfs (rw,nosuid,nodev)
none on /var/run type tmpfs (rw,nosuid,mode=0755)
none on /var/lock type tmpfs (rw,noexec,nosuid,nodev)
none on /lib/init/rw type tmpfs (rw,nosuid,mode=0755)
none on /var/lib/ureadahead/debugfs type debugfs (rw,relatime)
/dev/sda1 on /boot type ext2 (rw)
/dev/sdb1 on /storage type ext4 (rw,_netdev)
root@target:~#


Check the disk using the df command

#df -h

Filesystem Size Used Avail Use% Mounted on
/dev/mapper/server1-root
18G 839M 16G 5% /
none 243M 180K 242M 1% /dev
none 247M 0 247M 0% /dev/shm
none 247M 36K 247M 1% /var/run
none 247M 0 247M 0% /var/lock
none 247M 0 247M 0% /lib/init/rw
none 18G 839M 16G 5% /var/lib/ureadahead/debugfs
/dev/sda1 228M 17M 199M 8% /boot
/dev/sdb1 20G 172M 19G 1% /storage
root@target:~#

Test backup

You may backup the /storage/lun1.img file to a separate location. When you lose the original lun1.img file, you need to:

Recreate or recover the img file
Restart iscsitarget service on the target server
Restart open-iscsi service on the initiator
logout of the portal
discover the portal
login the portal again
unmount the mount
fdisk –l
mount the mount point
verify.
