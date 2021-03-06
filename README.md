The Optimistic File System
==========================
*The Optimistic File System (OptFS) is Linux ext4 variant that implements
Optimistic Crash Consistency, a new approach to crash consistency in journaling
file systems. OptFS improves performance for many workloads, sometimes by an
order of magnitude. OptFS provides strong consistency, equivalent to data
journaling mode of ext4.*

 For technical details, please read the [SOSP
 2013](http://sigops.org/sosp/sosp13/) paper [Optimistic Crash
 Consistency](http://www.cs.wisc.edu/adsl/Publications/optfs-sosp13.pdf).
 Please [cite this
 publication](http://research.cs.wisc.edu/adsl/Publications/optfs-sosp13.bib)
 if you use this work.

Please feel free to [contact me](http://cs.wisc.edu/~vijayc) with
any questions.

#### Description

The code is comprised of two parts:

* The modified Linux 3.2 kernel.

* The OptFS file-system module.

OptFS is referenced in the code as ext4bf (barrier-free version of ext4). The
module can be found at fs/ext4bf/.

#### Compiling and installing the kernel
* From the root kernel folder (where this README is found), do as root:
  
    <pre>mv kernel.config .config</pre>
    <pre>make -j4 && make modules && make modules_install && make install</pre>

* Fix up grub (/etc/default/grub) as required.
* Reboot into installed kernel.

#### Compiling and loading the OptFS module
Note: you must have already compiled and rebooted into the OptFS kernel before the following steps.
The following scripts run OptFS on a new disk partition (/dev/sdb1). If
required, you need to attach a second disk (/dev/sdb) to the machine, and
create a partition (/dev/sdb1) on the disk.

*   <pre>mkdir /mnt/mydisk</pre>

* Navigate to fs/ext4bf folder.

* Make the file system (on /dev/sdb1) using:
    
    <pre>sh mk-big.sh</pre>

   Warning: this creates a file system on /dev/sdb1, erasing all previous
   content. The drive has to be big enough to contain a 16 GB journal.

* Compile and load the module using
   
  <pre>sh load.sh</pre>

   This will result in an OptFS file system on /dev/sdb1, mounted at
   /mnt/mydisk.

#### Patches

We have included two patches: a full patch and an educational patch. The full
patch is the diff of OptFS kernel with a vanilla linux 3.2 kernel. Applying
this patch directly to a 3.2 kernel will result in the OptFS kernel.

The OptFS file-system module is modified on top of ext4 and jbd2. It contains
the code of both ext4 and jbd2. Diffing this against just ext4 will result in
a lot of redundant lines in the patch. Hence we have roughly merged ext4 and
jbd2 into a module, and diffed OptFS against this. This patch is presented as
optfs_patch_educational.

Note that optfs_patch_educational should never be applied to an actual system:
this will not work. It is provided so that users can easily view the
differences in code between OptFS and ext4, and is not meant to be applied
directly. 

#### Notes 

OptFS is built upon asynchronous durability notifications from the disk. Since
disks do not provide such functionality yet, we <b>emulate</b> such
notifications using durability timeouts in OptFS. This is basically a
guarantee from the disk that it will not cache a write request for longer than
a certain amount of time. This is set to be 30 seconds in OptFS.

This needs to be tuned for the disk that OptFS runs on. If the disk does cache
writes for more time than the OptFS setting, OptFS will not be able to
guarantee consistency in the case of a crash. 

#### Caveats 

This version of the code provides only *eventual durability* by default: every
OptFS (ext4bf) fsync() call behaves likes an osync(). Therefore, any unmodified
application running on ext4bf behaves as if every fsync() was replaced by an
osync(). This allows you to take any application and run it on OptFS to
determine the maximum performance you could potentially get. 

For applications requiring durability, please use the dsync() call. The code
can be modified fairly easily so that the default behavior is dsync() instead
of osync().

Note that the code is provided "as is": compiling and running the code will
require some tweaking based on the operating system environment. The file
system is only meant as a prototype and not meant for production use in any
way.
