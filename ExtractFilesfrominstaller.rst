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
