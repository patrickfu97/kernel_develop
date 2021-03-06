# procfs -- pseudo-filesystems
## Description
proc文件系统是虚拟文件系统，提供了内核数据结构的交互接口。通过proc文件系统可以得到很多内核才能看到的信息，比如cpuinfo，meminfo等。/proc下大部分文件都是只读的，只有少部分是可写入的，通过这些文件可以将信息传递到内核空间。
关于/proc的详细说明可以参考[man文档](https://www.man7.org/linux/man-pages/man5/proc.5.html)。
这里只关注对其下文件的监控。/proc是虚拟文件系统，是与内核交互的接口，所以并不存在真实的文件。实际上/proc下的内容并不是定时更新的，而是访问时，才会更新。因此要对其进行监控，只能通过定时read相应文件，然后比较两次的内容是否发生改变。

>[参考解释](https://unix.stackexchange.com/questions/74713/how-frequently-is-the-proc-file-system-updated-on-linux?rq=1):

>The information that you read from the proc filesystem is not stored on any media (not even in RAM), so there is nothing to update.

>The purpose of the proc file system is to allow userspace programs to obtain or set kernel data using the simple and familiar file system semantics (open, close, read, write, lseek), even though the data that is read or written doesn't reside on any media. This design decision was deemed better (e.g. human readable and easily scriptable) for getting and setting data whose format could not be specified in advance than implementing something such as ASN1 encoded OIDs, which also would have worked fine.

>The data that you see when you read from the proc filesystem is generated on-the-fly when you do a read from the begining of a file. That is, doing the read causes the data to be generated by a kernel callback function that is specific to the file you are reading. Doing an lseek to the begining of the file and reading again causes another call to the callback that generates the data again. Similarly, when you write to a writable file in the proc filesystem, a callback function is called that parses the input and sets kernel variables. The input data in it's raw form isn't stored.

>The above is just a slightly more verbose way of saying what Hauke Laging states so succinctly. I suggest that you accept his answer.

但其中也有一个是可以监控其变化的。[man文档](https://www.man7.org/linux/man-pages/man5/proc.5.html)中有提及。
>/proc/[pid]/mounts (since Linux 2.4.19)

>This file lists all the filesystems currently mounted in the process's mount namespace (see mount_namespaces(7)).  The format of this file is documented in fstab(5).

>Since kernel version 2.6.15, this file is pollable: after opening the file for reading, a change in this file (i.e., a filesystem mount or unmount) causes select(2) to mark the file descriptor as having an exceptional condition, and poll(2) and epoll_wait(2) mark the file as having a priority event (POLLPRI).  (Before Linux 2.6.30, a change in this file was indicated by the file descriptor being marked as readable for select(2), and being marked as having an error condition for poll(2) and epoll_wait(2).)

## Q&A

#### 1. 为什么inotify没办法监控/proc, /sys这些伪文件系统?
因为inotify使用的是poll来监控文件变化，而/proc，/sys这些属于伪文件系统，不能别监控。
>[man文档说明](http://man7.org/linux/man-pages/man7/inotify.7.html)

>Inotify reports only events that a user-space program triggers through the filesystem API.  As a result, it does not catch remote events that occur on network filesystems.  (Applications must fall back to polling the filesystem to catch such events.)  Furthermore, various pseudo-filesystems such as /proc, /sys, and /dev/pts are not monitorable with inotify.
---
