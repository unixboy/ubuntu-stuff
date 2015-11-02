How To Set Up an NFS Mount 
NFS, or Network File System, is a distributed filesystem protocol that allows you to mount remote directories on your server. This allows you to leverage storage space in a different location and to write to the same space from multiple servers easily. NFS works well for directories that will have to be accessed regularly.

In this guide, we'll cover how to configure NFS mounts on an Ubuntu 14.04 server.
Prerequisites

In this guide, we will be configuring directory sharing between two Ubuntu 14.04 servers. These can be of any size. For each of these servers, you will have to have an account set up with sudo privileges. You can learn how to configure such an account by following steps 1-4 in our initial setup guide for Ubuntu 14.04 servers.

For the purposes of this guide, we are going to refer to the server that is going to be sharing its directories the host and the server that will mount these directories as the client.

In order to keep these straight throughout the guide, I will be using the following IP addresses as stand-ins for the host and server values:

    Host: 1.2.3.4
    Client: 111.111.111.111

You should substitute the values above with your own host and client values.
Download and Install the Components

Before we can begin, we need to install the necessary components on both our host and client servers.

On the host server, we need to install the nfs-kernel-server package, which will allow us to share our directories. Since this is the first operation that we're performing with apt in this session, we'll refresh our local package index before the installation:

sudo apt-get update
sudo apt-get install nfs-kernel-server

Once these packages are installed, you can switch over to the client computer.

On the client computer, we're going to have to install a package called nfs-common, which provides NFS functionality without having to include the server components. Again, we will refresh the local package index prior to installation to ensure that we have up-to-date information:

sudo apt-get update
sudo apt-get install nfs-common

Create the Share Directory on the Host Server

We're going to experiment with sharing two separate directories during this guide. The first directory we're going to share is the /home directory that contains user data.

The second is a general purpose directory that we're going to create specifically for NFS so that we can demonstrate the proper procedures and settings. This will be located at /var/nfs.

Since the /home directory already exists, go ahead and start by creating the /var/nfs directory:

sudo mkdir /var/nfs

Now, we have a new directory designated specifically for sharing with remote hosts. However, the directory ownership is not ideal. We should give the user ownership to a user on our system named nobody. We should give the group ownership to a group on our system named nogroup as well.

We can do that by typing this command:

sudo chown nobody:nogroup /var/nfs

We only need to change the ownership on our directories that are used specifically for sharing. We wouldn't want to change the ownership of our /home directory, for instance, because it would cause a great number of problems for any users we have on our host server.
Configure the NFS Exports on the Host Server

Now that we have our directories created and assigned, we can dive into the NFS configuration file to set up the sharing of these resources.

Open the /etc/exports file in your text editor with root privileges:

sudo nano /etc/exports

The files that you see will have some comments that will show you the general structure of each configuration line. Basically, the syntax is something like:

directory_to_share       client(share_option1,...,share_optionN)

So we want to create a line for each of the directories that we wish to share. Since in this example or client has an IP of 111.111.111.111, our lines will look like this:

/home       111.111.111.111(rw,sync,no_root_squash,no_subtree_check)
/var/nfs    111.111.111.111(rw,sync,no_subtree_check)

We've explained everything here but the specific options we've enabled. Let's go over those now.

    rw: This option gives the client computer both read and write access to the volume.
    sync: This option forces NFS to write changes to disk before replying. This results in a more stable and consistent environment, since the reply reflects the actual state of the remote volume.
    nosubtreecheck: This option prevents subtree checking, which is a process where the host must check whether the file is actually still available in the exported tree for every request. This can cause many problems when a file is renamed while the client has it opened. In almost all cases, it is better to disable subtree checking.
    norootsquash: By default, NFS translates requests from a root user remotely into a non-privileged user on the server. This was supposed to be a security feature by not allowing a root account on the client to use the filesystem of the host as root. This directive disables this for certain shares.

When you are finished making your changes, save and close the file.

Next, you should create the NFS table that holds the exports of your shares by typing:

sudo exportfs -a

However, the NFS service is not actually running yet. You can start it by typing:

sudo service nfs-kernel-server start

This will make your shares available to the clients that you configured.
Create the Mount Points and Mount Remote Shares on the Client Server

Now that your host server is configured and making its directory shares available, we need to prep our client.

We're going to have to mount the remote shares, so let's create some mount points. We'll use the traditional /mnt as a starting point and create a directory called nfs under it to keep our shares consolidated.

The actual directories will correspond with their location on the host server. We can create each directory, and the necessary parent directories, by typing this:

sudo mkdir -p /mnt/nfs/home
sudo mkdir -p /mnt/nfs/var/nfs

Now that we have some place to put our remote shares, we can mount them by addressing our host server, which in this guide is 1.2.3.4, like this:

sudo mount 1.2.3.4:/home /mnt/nfs/home
sudo mount 1.2.3.4:/var/nfs /mnt/nfs/var/nfs

These should mount the shares from our host computer onto our client machine. We can double check this by looking at the available disk space on our client server:

df -h

Filesystem            Size  Used Avail Use% Mounted on
/dev/vda               59G  1.3G   55G   3% /
none                  4.0K     0  4.0K   0% /sys/fs/cgroup
udev                  2.0G   12K  2.0G   1% /dev
tmpfs                 396M  324K  396M   1% /run
none                  5.0M     0  5.0M   0% /run/lock
none                  2.0G     0  2.0G   0% /run/shm
none                  100M     0  100M   0% /run/user
1.2.3.4:/home          59G  1.3G   55G   3% /mnt/nfs/home

As you can see at the bottom, only one of our shares has shown up. This is because both of the shares that we exported are on the same filesystem on the remote server, meaning that they share the same pool of storage. In order for the Avail and Use% columns to remain accurate, only one share may be added into the calculations.

If you want to see all of the NFS shares that you have mounted, you can type:

mount -t nfs

1.2.3.4:/home on /mnt/nfs/home type nfs (rw,vers=4,addr=1.2.3.4,clientaddr=111.111.111.111)
1.2.3.4:/var/nfs on /mnt/nfs/var/nfs type nfs (rw,vers=4,addr=1.2.3.4,clientaddr=111.111.111.111)

This will show all of the NFS mounts that are currently accessible on your client machine.
Test NFS Access

You can test the access to your shares by writing something to your shares. You can write a test file to one of your shares like this:

sudo touch /mnt/nfs/home/test_home

Let's write test file to the other share as well to demonstrate an important difference:

sudo touch /mnt/nfs/var/nfs/test_var_nfs

Look at the ownership of the file in the mounted home directory:

ls -l /mnt/nfs/home/test_home

-rw-r--r-- 1 root   root      0 Apr 30 14:43 test_home

As you can see, the file is owned by root. This is because we disabled the root_squash option on this mount that would have written the file as an anonymous, non-root user.

On our other test file, which was mounted with the root_squash enabled, we will see something different:

ls -l /mnt/nfs/var/nfs/test_var_nfs

-rw-r--r-- 1 nobody nogroup 0 Apr 30 14:44 test_var_nfs

As you can see, this file was assigned to the "nobody" user and the "nogroup" group. This follows our configuration.
Make Remote NFS Directory Mounting Automatic

We can make the mounting of our remote NFS shares automatic by adding it to our fstab file on the client.

Open this file with root privileges in your text editor:

sudo nano /etc/fstab

At the bottom of the file, we're going to add a line for each of our shares. They will look like this:

1.2.3.4:/home    /mnt/nfs/home   nfs auto,noatime,nolock,bg,nfsvers=4,intr,tcp,actimeo=1800 0 0
1.2.3.4:/var/nfs    /mnt/nfs/var/nfs   nfs auto,noatime,nolock,bg,nfsvers=4,sec=krb5p,intr,tcp,actimeo=1800 0 0

The options that we are specifying here can be found in the man page that describes NFS mounting in the fstab file:

man nfs

This will automatically mount the remote partitions at boot (it may take a few moments for the connection to be made and the shares to be available).
Unmount an NFS Remote Share

If you no longer want the remote directory to be mounted on your system, you can unmount it easily by moving out of the share's directory structure and unmounting, like this:

cd ~
sudo umount /mnt/nfs/home
sudo umount /mnt/nfs/var/nfs

This will remove the remote shares, leaving only your local storage accessible:

df -h

Filesystem      Size  Used Avail Use% Mounted on
/dev/vda         59G  1.3G   55G   3% /
none            4.0K     0  4.0K   0% /sys/fs/cgroup
udev            2.0G   12K  2.0G   1% /dev
tmpfs           396M  320K  396M   1% /run
none            5.0M     0  5.0M   0% /run/lock
none            2.0G     0  2.0G   0% /run/shm
none            100M     0  100M   0% /run/user

As you can see, our NFS shares are no longer available as storage space.
