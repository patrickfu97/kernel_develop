# relayfs -- pseudo-filesystems

## Description
relay 是一种从 Linux 内核到用户空间的高效数据传输技术.
***



## Q&A

#### 1. 为什么inotify没办法监控/proc, /sys这些伪文件系统?
>[man文档说明](http://man7.org/linux/man-pages/man7/inotify.7.html)

>Inotify reports only events that a user-space program triggers through the filesystem API.  As a result, it does not catch remote events that occur on network filesystems.  (Applications must fall back to polling the filesystem to catch such events.)  Furthermore, various pseudo-filesystems such as /proc, /sys, and /dev/pts are not monitorable with inotify.
---
