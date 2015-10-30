	
FUSE and its access rights

lsof by default checks all mounted file systems including FUSE - file systems implemented in user space which have special access rights in Linux.

As you can see in this answer on Ask Ubuntu a mounted GVFS file system (special case of FUSE) is normally accessible only to the user which mounted it (the owner of gvfsd-fuse). Even root cannot access it. To override this restriction it is possible to use mount options allow_root and allow_other. The option must be also enabled in the FUSE daemon which is described for example in this answer ...but in your case you do not need to (and should not) change the access rights.

Excluding file systems from lsof

In your case lsof does not need to check the GVFS file systems so you can exclude the stat() calls on them using the -e option (or you can just ignore the waring):

lsof -e /run/user/1000/gvfs
Checking certain files by lsof

You are using lsof to get information about all processes running on your system and only then you filter the complete output using grep. If you want to check just certain files and the related processes use the -f option without a value directly following it then specify a list of files after the "end of options" separator --. This will be considerably faster.

lsof -e /run/user/1000/gvfs -f -- /tmp/report.csv
General solution

To exclude all mounted file systems on which stat() fails you can run something like this (in bash):

x=(); for a in $(mount | cut -d' ' -f3); do test -e "$a" || x+=("-e$a"); done
lsof "${x[@]}" -f -- /tmp/report.csv
Or to be sure to use stat() (test -e could be implemented a different way):

x=(); for a in $(mount | cut -d' ' -f3); do stat --printf= "$a" 2>/dev/null || x+=("-e$a"); done



Linux To determine if the number of open files is growing over a period of time, issue lsof to report the open files against a PID on a periodic basis. For example: 

lsof -p [PID] -r [interval in seconds, 1800 for 30 minutes] > lsof.out 

The output will provide you with all of the open files for the specified PID. You will be able to determine which files are opened and which files are growing over time. 


Alternately you can list the contents of the file descriptors as a list of symbolic links in the following directory, where you replace PID with the process ID. This is especially useful if you don't have access to the lsof command: 

ls -al /proc/PID/fd 

