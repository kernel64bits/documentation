# Inotify

https://www.linuxjournal.com/article/8478
https://man7.org/linux/man-pages/man7/inotify.7.html

Inotify is a system merged into the Linux kernel allowing a process to watch and receive event about filesystem changes.

```
The inotify API provides a mechanism for monitoring filesystem
       events.  Inotify can be used to monitor individual files, or to
       monitor directories.  When a directory is monitored, inotify will
       return events for the directory itself, and for files inside the
       directory.
```

On 25/10/2022 we hit a limit of the kernel configuration on the inotify system.
We couldn't create anymore containers on the host, because the user ROOT had reached the maximum limit of inotify instances.

Here are the relative configuration key of sysctl:
- fs.inotify.max_queued_events
  - Upper limit on the number of events that can be queued to the corresponding inotify instance (see inotify)
- fs.inotify.max_user_instances
  - Upper limit on the number of inotify instances that can be created per real user ID (see inotify)
- fs.inotify.max_user_watches
  - Upper limit on the number of watches that can be created per real user ID (see inotify)

On the LXC documentation website, we can see some recommanded values for a productio environment: https://linuxcontainers.org/lxd/docs/master/reference/server_settings/#server-settings

fs.inotify.max_queued_events -> 1048576
fs.inotify.max_user_instances -> 1048576
fs.inotify.max_user_watches -> 1048576

Right now, on one of the host, we got this values. They migth differ from one host to another if they have different kernel version (starting from Linux 5.X max_user_watches is dynamic).
fs.inotify.max_queued_events -> 16384
fs.inotify.max_user_instances -> 1024
fs.inotify.max_user_watches -> 1048576

On our bug, incrementing the value of `fs.inotify.max_user_instances` allowed us to create the containers. It seems we reached the maximum instances of inotify.