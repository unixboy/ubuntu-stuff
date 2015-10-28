How to View and Extract Files from rpm, deb, depot and msi Packages
  

Question: How do I view or extract the files that are bundled inside the packages of various operating system. For example, I would like to know how to view (and extract) the content of a rpm, or deb, or depot, or msi file.

Answer: You can use tools like rpm, rpm2cpio, ar, dpkg, tar, swlist, swcopy, lessmsi as explained below.

1. RPM package in Redhat / CentOS / Fedora

Listing the files from a RPM package using rpm -qlp

RPM stands for Red Hat package manager. The following example shows how to view the files available in a RPM package without extracting or installing the rpm package.

$ rpm -qlp ovpc-2.1.10.rpm
/usr/src/ovpc/-5.10.0
/usr/src/ovpc/ovpc-2.1.10/examples
/usr/src/ovpc/ovpc-2.1.10/examples/bin
/usr/src/ovpc/ovpc-2.1.10/examples/lib
/usr/src/ovpc/ovpc-2.1.10/examples/test
.
.
.
/usr/src/ovpc/ovpc-2.1.10/pcs
Explanation of the command: rpm -qlp ovpc-2.1.10.rpm

rpm — command
q — query the rpm file
l — list the files in the package
p — specify the package name
Extracting the files from a RPM package using rpm2cpio and cpio

RPM is a sort of a cpio archive. First, convert the rpm to cpio archive using rpm2cpio command. Next, use cpio command to extract the files from the archive as shown below.

$ rpm2cpio ovpc-2.1.10.rpm | cpio  -idmv
./usr/src/ovpc/-5.10.0
./usr/src/ovpc/ovpc-2.1.10/examples
./usr/src/ovpc/ovpc-2.1.10/examples/bin
./usr/src/ovpc/ovpc-2.1.10/examples/lib
./usr/src/ovpc/ovpc-2.1.10/examples/test
.
.
.
./usr/src/ovpc/ovpc-2.1.10/pcs

$ ls .
usr
2. Deb package in Debian

deb is the extension of Debian software package format. *.deb is also used in other distributions that are based on Debian. (for example: Ubuntu uses *.deb)

Listing the files from a debian package using dpkg -c

dpkg is the package manager for debian. So using dpkg command you can list and extract the packages, as shown below.


 
To view the content of *.deb file:

$ dpkg -c ovpc_1.06.94-3_i386.deb
dr-xr-xr-x root/root         0 2010-02-25 10:54 ./                                                                                          
dr-xr-xr-x root/root         0 2010-02-25 10:54 ./ovpc/                                                                                    
dr-xr-xr-x root/root         0 2010-02-25 10:54 ./ovpc/pkg/                                                                            
dr-xr-xr-x root/root         0 2010-02-25 10:54 ./ovpc/pkg/lib/                                                                 
dr-xr-xr-x root/root         0 2010-02-25 10:48 ./ovpc/pkg/lib/header/                                                      
-r-xr-xr-x root/root       130 2009-10-29 17:06 ./ovpc/pkg/lib/header/libov.so                                   
.
.
.

-r-xr-xr-x root/root       131 2009-10-29 17:06 ./ovpc/pkg/etc/conf                                   
dr-xr-xr-x root/root         0 2010-02-25 10:54 ./ovpc/pkg/etc/conf/log.conf   
Extracting the files from a debian package using dpkg -x

Use dpkg -x to extract the files from a deb package as shown below.

$ dpkg -x  ovpc_1.06.94-3_i386.deb /tmp/ov
$ ls /tmp/ov
ovpc
DEB files are ar archives, which always contains the three files — debian-binary, control.tar.gz, and data.tar.gz. We can use ar command and tar command to extract and view the files from the deb package, as shown below.

First, extract the content of *.deb archive file using ar command.

$ ar -vx ovpc_1.06.94-3_i386.deb
x - debian-binary
x - control.tar.gz
x - data.tar.gz
$
Next, extract the content of data.tar.gz file as shown below.

$ tar -xvzf data.tar.gz 
./                                                                             
./ovpc/                                                                         
./ovpc/pkg/                                                                     
./ovpc/pkg/lib/                                                             
./ovpc/pkg/lib/header/                                                      
./ovpc/pkg/lib/header/libov.so                                   
.
.
./ovpc/pkg/etc/conf                                   
./ovpc/pkg/etc/conf/log.con
3. Depot package in HP-UX

Listing the files from a depot package using tar and swlist

DEPOT file is a HP-UX Software Distributor Catalog Depot file. HP-UX depots are just a tar file, with some additional information as shown below.

$ tar -tf ovcsw_3672.depot
OcswServer/MGR/etc/
OcswServer/MGR/etc/opt/
OcswServer/MGR/etc/opt/OV/
OcswServer/MGR/etc/opt/OV/share/
OcswServer/MGR/etc/opt/OV/share/conf/
OcswServer/MGR/etc/opt/OV/share/conf/OpC/
OcswServer/MGR/etc/opt/OV/share/conf/OpC/opcctrlovw/
swlist is a HP-UX command which is used to display the information about the software. View the content of the depot package as shown below using swlist command.

$ swlist -l file -s /root/ovcsw_3672.depot
# Initializing...
# Contacting target "osgsw"...
#
# Target:  osgsw:/root/ovcsw_3672.depot
#

# OcswServer			8.50.000       Ocsw  Server product
# OcswServer.MGR     		9.00.140       Ocs Server Ovw
  /etc
  /etc/opt
  /etc/opt/OV
  /etc/opt/OV/share
  /etc/opt/OV/share/conf
  /etc/opt/OV/share/conf/OpC
Extracting the files from a depot package using swcopy

Swcopy command copies or merges software_selections from a software source to one or more software depot target_selections. Using uncompress option in swcopy, you can extract the files from a depot software package.

$ swcopy -x uncompress_files=true -x enforce_dependencies=false -s /root/ovcsw_3672.depot \* @ /root/extracted/
$ ls /root/extracted
MGR	catalog	 osmsw.log
$
Since depot files tar files, you can extract using normal tar extraction as shown below.

$ tar -xvf filename
4. MSI in Windows

Microsoft installer is an engine for the installation, maintenance, and removal of software on windows systems.

Listing the files from a MSI package using lessmsi

The utility called lessmsi.exe is used to view the files from the msi packages with out installing. The same utility is also used to extract the msi package. Select the msi which you want to view the content. lessmsi will list the files available in msi.

Extracting the files from a MSI package using msiexec

Windows Installer Tool (Msiexec.exe) is used to extract the files from the MSI package. It can open a MSI package in “Administrator” installation mode, where it can extract the files without performing the install as shown below.

C:\>msiexec /a ovcsw_3672.msi /qb TARGETDIR="C:\ovcsw"




For RPMs you need two command line utilities, rpm2cpio and cpio. Extracting the contents of the RPM package is a one step process:

rpm2cpio mypackage.rpm | cpio -vid
If you just need to list the contents of the package without extracting them, use the following:

rpm2cpio mypackage.rpm | cpio -vt
The -v option is used in order to get verbose output to the stdout. If you don’t need it, you can safely omit this switch. For more information about the cpio options, please refer to the cpio(1) manual page.

DEB
DEB files are ar archives, which contain three files:

debian-binary
control.tar.gz
data.tar.gz
As you might have already guessed, the needed archived files exist in data.tar.gz. It is also obvious that unpacking this file is a two-step process.

First, extract the aforementioned three files from the DEB file (ar archive):

ar vx mypackage.deb
Then extract the contents of data.tar.gz using tar:

tar -xzvf data.tar.gz
Or, if you just need to get a listing of the files:

tar -tzvf data.tar.gz
Again the -v option in both ar and tar is used in order to get verbose output. It is safe not to use it. For more information, read the man pages: tar(1) and ar(1).

If anyone knows a one step process to extract the contents of the data.tar.gz, I’d be very interested in it!

Update 1
As Jon suggested in the comment area, the contents of data.tar.gz can be extracted from the DEB package in a one step process as shown below:

ar p mypackage.deb data.tar.gz | tar zx
Update 2 – Debian 8
As Vlad suggested in the comments below, starting with Debian 8, data.tar.gz inside .deb packages has been replaced with data.tar.xz (the xz format, which is based on LZMA2 offers better compression). So, if you are using Debian 8 or newer, the one-liner should be updated to something like this:

ar p mypackage.deb data.tar.xz | unxz | tar x
That will do it.


Restore original configuration files from RPM packages


By default, when the user installs software through the RPM Package Manager or through YUM, usually, the software’s configuration files included in the RPM do not replace the existing configuration files on the filesystem, but, if they differ from those that currently exist, they are saved with the rpmnew extension. In case the rpm is already installed and is the latest version, the quickest way to get the original configuration file back is to uninstall and install the package again. Today, while on CentOS 6.2, I needed to restore the original /etc/sysctl.conf file, which is part of the initscripts package. In this case, uninstalling initscripts was out of the question as it would also remove half of the installed packages due to dependencies. So, I grabbed the chance to figure out and document what would be the quickest and easiest way to restore /etc/sysctl.conf, excluding downloading the package itself and extract the RPM contents. Fortunately, as soon as I opened yum’s man page and having spotted the new reinstall command, the solution was quite obvious.

For completeness, I hereby document the whole procedure that involves the verification and restoration of the original /etc/sysctl.conf hoping that new users might find these notes helpful.

First of all, I needed to know whether the /etc/sysctl.conf I had on my box differed from the original one. But, before doing that, I had to know which RPM package had installed that file. So, I used the rpm command to query this file:

# rpm -qf /etc/sysctl.conf
initscripts-9.03.27-1.el6.centos.1.i686
So, the initscripts package had installed /etc/sysctl.conf.

Then I verified the initscripts package:

# rpm -V initscripts
S.5....T.  c /etc/sysconfig/init
S.5....T.  c /etc/sysctl.conf

 
According to the following table:

S file Size differs
M Mode differs (includes permissions and file type)
5 MD5 sum differs
D Device major/minor number mismatch
L readLink(2) path mismatch
U User ownership differs
G Group ownership differs
T mTime differs
P caPabilities differ
the attributes: size, MD5 checksum and the modification time of /etc/sysctl.conf that existed on my system differed from the attributes of the original file.

Since I had no idea the exact changes I had made to that file at some earlier time, I needed to restore the original and re-modify it from scratch. The new yum “reinstall” command could be used to to do this quite easily.

First, I kept a copy of the current file:

# mv /etc/sysctl.conf /etc/sysctl.conf.modified
Then I reinstalled initscripts using YUM’s reinstall command:

# yum reinstall initscripts
Loaded plugins: downloadonly, fastestmirror, priorities
Setting up Reinstall Process
Loading mirror speeds from cached hostfile
 * base: ftp.ntua.gr
 * epel: ftp.ntua.gr
 * extras: ftp.ntua.gr
 * ius: mirror.rackspace.co.uk
 * updates: centosr3.centos.org
6 packages excluded due to repository priority protections
Resolving Dependencies
--> Running transaction check
---> Package initscripts.i686 0:9.03.27-1.el6.centos.1 will be reinstalled
--> Finished Dependency Resolution

Dependencies Resolved

=============================================================================================================================================================
 Package                              Arch                          Version                                           Repository                        Size
=============================================================================================================================================================
Reinstalling:
 initscripts                          i686                          9.03.27-1.el6.centos.1                            updates                          934 k

Transaction Summary
=============================================================================================================================================================
Reinstall     1 Package(s)

Total download size: 934 k
Installed size: 5.4 M
Is this ok [y/N]: y
Downloading Packages:
initscripts-9.03.27-1.el6.centos.1.i686.rpm                                                                                           | 934 kB     00:02
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : initscripts-9.03.27-1.el6.centos.1.i686                                                                                                   1/1

Installed:
  initscripts.i686 0:9.03.27-1.el6.centos.1

Complete!
Verify the initscripts package again:

# rpm -V initscripts
S.5....T.  c /etc/sysconfig/init
No verification errors for /etc/sysctl.conf. Note that reinstalling the package did not touch the /etc/sysconfig/init file. It has been mentioned previously that rpm packages do not overwrite existing configuration files.



What Is An RPM Package?

 
In simple terms, an RPM package is an advanced form of a container for other files. Generally, it includes:

The program to be installed plus all the necessary files that accompany this program.
Information about the program and the RPM package itself.
Information about the program’s dependencies, which means info about what other software needs to be installed, so your program to function correctly in the system.
Information about potential conflicts between the program and other software that is currently installed in the system.
Actions that need to be performed when the program is installed/upgraded/removed.
But, this is enough with the theory. For more information, refer to the links at the Further Reading section of this article.

Prerequisites
The following are needed in order to build RPM Packages.

The development tools. All the utilities that are needed to compile a program, including the compiler itself.
A SPEC file for the particular program you want to create an RPM package for. A SPEC file contains all the information regarding the program’s details, its dependencies, the compilation options etc. For more info on writing SPEC files, refer to the links at the Further Reading section of this article.
Things You Need To Do Once
There are a couple of thing you need to do before starting building your RPMs. These mainly include the installation of the core development tools and the creation of the building environment for your user.

Install the core development tools using YUM. As root:

# yum groupinstall "Development Tools"
Next, create the building environment for your user. Fortunately, Fedora includes some neat utilities that greatly simplify this procedure. First, use YUM to install them (as root):

# yum install rpmdevtools
Then, create the directory structure in your home directory by issuing the command (as a user):

$ rpmdev-setuptree
That’s it.

Build That RPM!
Provided that you have a SPEC file for your program, you can build the binary RPM package by issuing the command:

$ rpmbuild -bb --clean myprogram.spec
If you need to build the package for a different architecture, you can set the --target option, like in the example below:

$ rpmbuild -bb --clean --target i686 myprogram.spec
Please note that you should never build RPM packages using root.

Dependencies
Some programs may need additional development libraries in order to be compiled. You can use YUM to install these needed libraries (-devel packages) or programs. If the operation finishes succesfully, you’ll find your RPM package in the ~/rpmbuild/RPMS/ directory.

  Fedora Core Developer’s Guide

https://docs.fedoraproject.org/en-US/Fedora_Draft_Documentation/0.1/html/RPM_Guide/

https://fedoraproject.org/wiki/Packaging:ScriptletSnippets?rd=Packaging/ScriptletSnippets

http://www.rpm.org/max-rpm-snapshot/

