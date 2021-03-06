
FUSD: A Linux Framework for User-Space Devices
----------------------------------------------

**Welcome to FUSD!**

This is FUSD snapshot 20110401, released 18 January 2012. This fork is based
on the found on the xiph.org SVN tracker. ( http://svn.xiph.org/trunk/fusd ) 
They seems to no longer update this tool (since 11 January 2007) and since it
longer compile with recent Linux kernel (at around 2.6.21) and since I need
it in personal project, I ported it to newer version (current version is 2.6.32).
It is currently no officialy supporting newer kernel, but changes are currently 
going on to support newer kernel up to 4.15. It is currently building under this
line, but don't expect it to work.
 

Some feature are still missing missing or buggy form the Xiph version (due to
changes on the kernel source tree), but it is still usable.

The official URL for this fork is:

http://github.com/Godzil/fusd

There is extensive documentation available in the 'doc' directory.
The FUSD User Manual is available in PDF, Postscript, and HTML format.
Most of this documentation dates from earlier versions of fusd; until
it is fully updated, it may not cover all features that exist in the
current version of fusd.

FUSD is free and open source software, released under a BSD-style
license. See the file 'LICENSE' for details.


QUICK START GUIDE
=================

Instructions for the impatient:

1. Make sure you're using a system running Linux 2.6.x with udev; this
version of fusd is incompatable with the now-deprecated devfs. If the
kernel is a packaged version from a distribution, also verify any
optional packages needed for building new kernel modules are also
installed.

2. 'make ; make install' builds everything including examples, then
installs the libraries, includes and kernel module.

3. Update the udev configuration (usually in /etc/udev/rules.d/) to
include the following rule:

         fusd device
         SUBSYSTEM=="fusd",                      NAME="fusd/%k"

    After updating, restart udevd (skill udevd; udevd -d).

4. Insert the FUSD kernel module (`modprobe kfusd`)

5. Verify the fusd devices /dev/fusd/status and /dev/fusd/control
exist. If the modprobe succeeds but no fusd devices appear,
double-check the udev rule config change and make sure udevd restarted
successfully. The kfusd kernel module must be inserted after udev has
been correctly configured and restarted.

6. Try running the `helloworld` example program (examples/helloworld).
When helloworld is running, `cat /dev/helloworld` should return
`Hello, world!`.

7. For more information, read the User's Manual in the 'doc' directory.

WHAT IS FUSD?
=============

FUSD (pronounced "fused") is a Linux framework for proxying device
file callbacks into user-space, allowing device files to be
implemented by daemons instead of kernel code. Despite being
implemented in user-space, FUSD devices can look and act just like any
other file under /dev which is implemented by kernel callbacks.

A user-space device driver can do many of the things that kernel
drivers can't, such as perform a long-running computation, block while
waiting for an event, or read files from the file system. Unlike
kernel drivers, a user-space device driver can use other device
drivers--that is, access the network, talk to a serial port, get
interactive input from the user, pop up GUI windows, or read from
disks. User-space drivers implemented using FUSD can be much easier to
debug; it is impossible for them to crash the machine, are easily
traceable using tools such as gdb, and can be killed and restarted
without rebooting. FUSD drivers don't have to be in C--Perl, Python,
or any other language that knows how to read from and write to a file
descriptor can work with FUSD. User-space drivers can be swapped out,
whereas kernel drivers lock physical memory.

FUSD drivers are conceptually similar to kernel drivers: a set of
callback functions called in response to system calls made on file
descriptors by user programs. FUSD's C library provides a device
registration function, similar to the kernel's devfs_register_chrdev()
function, to create new devices. fusd_register() accepts the device
name and a structure full of pointers. Those pointers are callback
functions which are called in response to certain user system
calls--for example, when a process tries to open, close, read from, or
write to the device file. The callback functions should conform to
the standard definitions of POSIX system call behavior. In many ways,
the user-space FUSD callback functions are identical to their kernel
counterparts.

The proxying of kernel system calls that makes this kind of program
possible is implemented by FUSD, using a combination of a kernel
module and cooperating user-space library. The kernel module
implements a character device, /dev/fusd, which is used as a control
channel between the two. fusd_register() uses this channel to send a
message to the FUSD kernel module, telling the name of the device the
user wants to register. The kernel module, in turn, registers that
device with the kernel proper using devfs. devfs and the kernel don't
know anything unusual is happening; it appears from their point of
view that the registered devices are simply being implemented by the
FUSD module.

Later, when kernel makes a callback due to a system call (e.g. when
the character device file is opened or read), the FUSD kernel module's
callback blocks the calling process, marshals the arguments of the
callback into a message and sends it to user-space. Once there, the
library half of FUSD unmarshals it and calls whatever user-space
callback the FUSD driver passed to fusd_register(). When that
user-space callback returns a value, the process happens in reverse:
the return value and its side-effects are marshaled by the library
and sent to the kernel. The FUSD kernel module unmarshals this
message, matches it up with a corresponding outstanding request, and
completes the system call. The calling process is completely unaware
of this trickery; it simply enters the kernel once, blocks, unblocks,
and returns from the system call---just as it would for any other
blocking call.

One of the primary design goals of FUSD is stability. It should
not be possible for a FUSD driver to corrupt or crash the kernel,
either due to error or malice. Of course, a buggy driver itself may
corrupt itself (e.g., due to a buffer overrun). However, strict error
checking is implemented at the user-kernel boundary which should
prevent drivers from corrupting the kernel or any other user-space
process---including the errant driver's own clients, and other FUSD
drivers.

For more information, please see the comprehensive documentation in
the 'doc' directory.
 
> Jeremy Elson <jelson@circlemud.org>                              <br>
> August 19, 2003                                                  <br>

> updated,<br>
> Monty <monty@xiph.org>                                           <br>
> January 11, 2007                                                 <br>

> Updated,                                                         <br>
> Godzil <godzil@godzil.net>                                       <br>
> March 01, 2011 / January 18, 2012 (public release on github)
