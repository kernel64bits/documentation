

This documentation is not yet ready
# cgroups
## Introduction
## Theoretical part
### Presentation of cgroups
#### Vocabulary
- subsystem: a subsystem represents a single resource (cpu time, memory, disks I/O, network traffic, ...)
- cgroup controller, ressource controller, or controller: Synonymous with cgroup subsystem. 
- task: a system process. A single task may be in multiple cgroups, as long as each of those cgroups is in a different hierarchy
- hiearchy: cgroups are hierarchical and child cgroups inherit certain attributes from their parents. There may be several separate cgroup hierarchies because each hierarchy is attached to one or several subsystems (ie resource)

#### What are cgroups used for
Using the control groups (`cgroups`) kernel functionality, you can control resource usage of applications to use them more efficiently.

You can use `cgroups` for the following tasks:

- Setting limits for system resource allocation.
- Prioritizing the allocation of hardware resources to specific processes.
- Isolating certain processes from obtaining hardware resources.

Using the _control groups_ Linux kernel feature, you can organize processes into hierarchically ordered groups - `cgroups`. You define the hierarchy (control groups tree) by providing structure to `cgroups` virtual file system, mounted by default on the `/sys/fs/cgroup/` directory.

The `systemd` service manager uses `cgroups` to organize all units and services that it governs. Manually, you can manage the hierarchies of `cgroups` by creating and removing sub-directories in the `/sys/fs/cgroup/` directory.

The resource controllers in the kernel then modify the behavior of processes in `cgroups` by limiting, prioritizing or allocating system resources, of those processes. These resources include the following:

- CPU time
- Memory
- Network bandwidth
- Combinations of these resources

The primary use case of `cgroups` is aggregating system processes and dividing hardware resources among applications and users. This makes it possible to increase the efficiency, stability, and security of your environment.

[Source](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/managing_monitoring_and_updating_the_kernel/setting-limits-for-applications_managing-monitoring-and-updating-the-kernel)
## cgroup v1 and cgroup v2

### Why were cgroups v2 created ?
https://medium.com/some-tldrs/tldr-understanding-the-new-control-groups-api-by-rami-rosen-980df476f633
The problem with cgroup 1 TODO
#### Simple architecture (TODO)
cgroup v1 has independent trees for each of controllers. eg. a process can join group "foo” for CPU (`/sys/fs/cgroup/cpu/foo` ) while joining group “bar” for memory ( `/sys/fs/cgroup/memory/bar` ). While this design seemed to provide good flexibility, it wasn’t proved to be useful in practice.

cgroup v2 focuses on simplicity: `/sys/fs/cgroup/cpu/$GROUPNAME` and `/sys/fs/cgroup/memory/$GROUPNAME` in v1 are now unified as `/sys/fs/cgroup/$GROUPNAME` , and a process can no longer join different groups for different controllers. If the process joins `foo` ( `/sys/fs/cgroup/foo` ), all controllers enabled for `foo` will take the control of the process.

https://medium.com/nttlabs/cgroup-v2-596d035be4d7
#### Differences
##### Control groups version 1
_Control groups version 1_ (`cgroups-v1`) provide a per-resource controller hierarchy. This means that each resource (such as CPU, memory, or I/O) has its own control group hierarchy. You can combine different control group hierarchies in a way that one controller can coordinate with another in managing their respective resources. However, when the two controllers belong to different process hierarchies, proper coordination is limited.

The `cgroups-v1` controllers were developed across a large time span and as a result, the behavior and naming of their control files is not uniform.

https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/managing_monitoring_and_updating_the_kernel/setting-limits-for-applications_managing-monitoring-and-updating-the-kernel
##### Control groups version 2
_Control groups version 2_ (`cgroups-v2`) provide a single control group hierarchy against which all resource controllers are mounted.

The control file behavior and naming is consistent among different controllers.
https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/managing_monitoring_and_updating_the_kernel/setting-limits-for-applications_managing-monitoring-and-updating-the-kernel
## Compatibility
Systemd has 3 kinds of cgroup hierarchies:
- **Unified** (default on Ubuntu 22): this is the simplest mode, and exposes a pure cgroup v2 logic. In this mode `/sys/fs/cgroup` is the only mounted cgroup API file system and all available controllers are exclusively exposed through it.
- **Legacy** — this is the traditional cgroup v1 mode. In this mode the various controllers each get their own cgroup file system mounted to `/sys/fs/cgroup/<controller>/`. On top of that systemd manages its own cgroup hierarchy for managing purposes as `/sys/fs/cgroup/systemd/`. 
- **Hybrid** (default on Ubuntu 18): This is a hybrid between the unified and legacy mode. It's set up mostly like legacy, except that there's also an additional hierarchy `/sys/fs/cgroup/unified/` that contains the cgroup v2 hierarchy. (Note that in this mode the unified hierarchy won't have controllers attached, the controllers are all mounted as separate hierarchies as in legacy mode, i.e. `/sys/fs/cgroup/unified/` is purely and exclusively about core cgroup v2 functionality and not about resource management.) In this mode compatibility with cgroup v1 is retained while some cgroup v2 features are available too. This mode is a stopgap. Don't bother with this too much unless you have too much free time.
[Source](https://github.com/systemd/systemd/blob/main/docs/CGROUP_DELEGATION.md#three-different-tree-setups-)

### Can I use both cgroup v1 and v2 ?
yes but TODO

## Practical part
### Visualize cgroups
```bash
ubuntu@server:~$ sudo systemctl status
● server
    State: degraded
     Jobs: 0 queued
   Failed: 5 units
    Since: Mon 2023-09-04 13:19:42 UTC; 4 months 0 days ago
   CGroup: /
           ├─26560 /usr/bin/SCREEN -dmS init-<container> /usr/bin/lxc-start --logpriority=DEBUG --logfile=/home/containers/<container>/init.log -n <container> -f /home/containers/<container>/config -F
           ├─user.slice
           │ └─user-2023.slice
           │   ├─session-7161.scope
           │   │ ├─19756 sshd: ubuntu [priv]
           │   │ ├─19833 sshd: ubuntu@pts/8
           │   │ ├─19835 -bash
           │   │ ├─19862 sudo systemctl status
           │   │ ├─19863 systemctl status
           │   │ └─19864 pager
           │   └─user@2023.service
           │     └─init.scope
           │       ├─19758 /lib/systemd/systemd --user
           │       └─19759 (sd-pam)
           ├─init.scope
           │ └─1 /sbin/init
           ├─system.slice
           │ ├─irqbalance.service
           ...
           └─lxc
             └─<container>
               ├─init.scope
               │ └─26572 /sbin/init
               └─system.slice
                 ├─ssh.service
                 │ └─26907 /usr/sbin/sshd -D
                 ├─rsyslog.service
                 │ └─26895 /usr/sbin/rsyslogd -n
                 ...
```
#### Different hierarchies
```bash
ubuntu@ns3208681.ip-141-95-144:~$ ll /sys/fs/cgroup/
total 0
drwxr-xr-x 15 root root 380 Oct 17 09:44 ./
drwxr-xr-x  8 root root   0 Oct 17 09:44 ../
dr-xr-xr-x  4 root root   0 Oct 17 09:44 blkio/
lrwxrwxrwx  1 root root  11 Oct 17 09:44 cpu -> cpu,cpuacct/
dr-xr-xr-x  4 root root   0 Oct 17 09:44 cpu,cpuacct/
lrwxrwxrwx  1 root root  11 Oct 17 09:44 cpuacct -> cpu,cpuacct/
dr-xr-xr-x  2 root root   0 Oct 17 09:44 cpuset/
dr-xr-xr-x  4 root root   0 Oct 17 09:44 devices/
dr-xr-xr-x  3 root root   0 Oct 17 09:44 freezer/
dr-xr-xr-x  5 root root   0 Oct 17 09:44 memory/
dr-xr-xr-x  2 root root   0 Oct 17 09:44 misc/
lrwxrwxrwx  1 root root  16 Oct 17 09:44 net_cls -> net_cls,net_prio/
dr-xr-xr-x  2 root root   0 Oct 17 09:44 net_cls,net_prio/
lrwxrwxrwx  1 root root  16 Oct 17 09:44 net_prio -> net_cls,net_prio/
dr-xr-xr-x  2 root root   0 Oct 17 09:44 perf_event/
dr-xr-xr-x  4 root root   0 Oct 17 09:44 pids/
dr-xr-xr-x  2 root root   0 Oct 17 09:44 rdma/
dr-xr-xr-x  6 root root   0 Oct 17 09:44 systemd/
dr-xr-xr-x  5 root root   0 Oct 17 09:44 unified/
```
#### Inside a given hiearchy
```bash
ubuntu@server:/sys/fs/cgroup/memory$ ll
total 0
dr-xr-xr-x  6 root root   0 Sep  4 13:19 ./
drwxr-xr-x 15 root root 380 Sep  4 13:19 ../
-rw-r--r--  1 root root   0 Sep  5 01:22 cgroup.clone_children
--w--w--w-  1 root root   0 Sep  5 01:22 cgroup.event_control
-rw-r--r--  1 root root   0 Jan  3 16:32 cgroup.procs
-r--r--r--  1 root root   0 Sep  5 01:22 cgroup.sane_behavior
drwxr-xr-x  3 root root   0 Oct 19 12:49 lxc/
-rw-r--r--  1 root root   0 Sep  5 01:22 memory.failcnt
--w-------  1 root root   0 Sep  5 01:22 memory.force_empty
-rw-r--r--  1 root root   0 Sep  5 01:22 memory.kmem.failcnt
-rw-r--r--  1 root root   0 Sep  5 01:22 memory.kmem.limit_in_bytes
-rw-r--r--  1 root root   0 Sep  5 01:22 memory.kmem.max_usage_in_bytes
-r--r--r--  1 root root   0 Sep  5 01:22 memory.kmem.slabinfo
-rw-r--r--  1 root root   0 Sep  5 01:22 memory.kmem.tcp.failcnt
-rw-r--r--  1 root root   0 Sep  5 01:22 memory.kmem.tcp.limit_in_bytes
-rw-r--r--  1 root root   0 Sep  5 01:22 memory.kmem.tcp.max_usage_in_bytes
-r--r--r--  1 root root   0 Sep  5 01:22 memory.kmem.tcp.usage_in_bytes
-r--r--r--  1 root root   0 Sep  5 01:22 memory.kmem.usage_in_bytes
-rw-r--r--  1 root root   0 Sep  4 13:19 memory.limit_in_bytes
-rw-r--r--  1 root root   0 Sep  5 01:22 memory.max_usage_in_bytes
-rw-r--r--  1 root root   0 Sep  5 01:22 memory.memsw.failcnt
-rw-r--r--  1 root root   0 Sep  5 01:22 memory.memsw.limit_in_bytes
-rw-r--r--  1 root root   0 Sep  5 01:22 memory.memsw.max_usage_in_bytes
-r--r--r--  1 root root   0 Sep  5 01:22 memory.memsw.usage_in_bytes
-rw-r--r--  1 root root   0 Sep  5 01:22 memory.move_charge_at_immigrate
-r--r--r--  1 root root   0 Sep  5 01:22 memory.numa_stat
-rw-r--r--  1 root root   0 Sep  5 01:22 memory.oom_control
----------  1 root root   0 Sep  5 01:22 memory.pressure_level
-rw-r--r--  1 root root   0 Sep  5 01:22 memory.soft_limit_in_bytes
-r--r--r--  1 root root   0 Sep  5 01:22 memory.stat
-rw-r--r--  1 root root   0 Sep  5 01:22 memory.swappiness
-r--r--r--  1 root root   0 Sep  5 01:22 memory.usage_in_bytes
-rw-r--r--  1 root root   0 Sep  4 13:19 memory.use_hierarchy
-rw-r--r--  1 root root   0 Sep  5 01:22 notify_on_release
-rw-r--r--  1 root root   0 Sep  5 01:22 release_agent
drwxr-xr-x 66 root root   0 Sep  4 13:19 system.slice/
-rw-r--r--  1 root root   0 Sep  5 01:22 tasks # List of PIDs attached to this cgroup
drwxr-xr-x  9 root root   0 Sep  4 13:23 user/
```

#### Link with LXC containers
Each container has it own cgroup for each hierarchy. All the container PIDs are attached to the container cgroup
```bash
ubuntu@server:/sys/fs/cgroup/memory$ cat lxc/<container>/tasks
26572
26871
26889
26894
26895
26896
26897
26898
26899
26900
26901
26902
26903
26904
26905
26906
26907
27012
27013
27014


root     26560  0.0  0.0  32036  2784 ?        Ss   16:30   0:00 /usr/bin/SCREEN -dmS init-container /usr/bin/lxc-start --logpriority=DEBUG --logfile
root     26562  0.0  0.0  57180  5276 pts/9    Ss+  16:30   0:00  \_ /usr/bin/lxc-start --logpriority=DEBUG --logfile=/home/containers/<container>/init
root     26572  0.0  0.0  77124  8144 ?        Ss   16:30   0:00      \_ /sbin/init
root     26871  0.0  0.0 108700 15300 ?        S<s  16:30   0:00          \_ /lib/systemd/systemd-journald
systemd+ 26889  0.0  0.0  70620  5164 ?        Ss   16:30   0:00          \_ /lib/systemd/systemd-resolved
root     26894  0.0  0.0  30116  3012 ?        Ss   16:30   0:00          \_ /usr/sbin/cron -f
uuidd    26895  0.0  0.0 336748  4032 ?        Ssl  16:30   0:00          \_ /usr/sbin/rsyslogd -n
root     26896  0.0  0.0  62008  5576 ?        Ss   16:30   0:00          \_ /lib/systemd/systemd-logind
syslog   26897  0.0  0.0  47640  4088 ?        Ss   16:30   0:00          \_ /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --syste
root     26902  0.0  0.0  14780  2224 pts/0    Ss+  16:30   0:00          \_ /sbin/agetty -o -p -- \u --noclear --keep-baud pts/0 115200,38400,9600 vt220
root     26903  0.0  0.0  14780  2324 ?        Ss+  16:30   0:00          \_ /sbin/agetty -o -p -- \u --noclear --keep-baud pts/3 115200,38400,9600 vt220
root     26904  0.0  0.0  14780  2392 ?        Ss+  16:30   0:00          \_ /sbin/agetty -o -p -- \u --noclear --keep-baud pts/2 115200,38400,9600 vt220
root     26905  0.0  0.0  14780  2328 ?        Ss+  16:30   0:00          \_ /sbin/agetty -o -p -- \u --noclear --keep-baud pts/1 115200,38400,9600 vt220
root     26906  0.0  0.0  14780  2316 pts/10   Ss+  16:30   0:00          \_ /sbin/agetty -o -p -- \u --noclear --keep-baud console 115200,38400,9600 vt22
root     26907  0.0  0.0  72304  5688 ?        Ss   16:30   0:00          \_ /usr/sbin/sshd -D
root     27012  0.0  0.0  67388  3532 ?        Ss   16:30   0:00          \_ /usr/lib/postfix/sbin/master -w
lxd      27013  0.0  0.0  69448  4788 ?        S    16:30   0:00              \_ pickup -l -t fifo -u
lxd      27014  0.0  0.0  69496  4840 ?        S    16:30   0:00              \_ qmgr -l -t fifo -u
```

```bash
ubuntu@server:/sys/fs/cgroup/memory/lxc/<container>$ cat memory.limit_in_bytes
107374182400 # 107374182400 = 100 * 1024^3 = 100 GiB

ubuntu@server:/sys/fs/cgroup/memory/lxc/<container>$ cat /home/containers/<container>/config
[...]

lxc.include = /usr/share/lxc/config/common.conf

lxc.cgroup.cpu.shares = 400
lxc.cgroup.memory.soft_limit_in_bytes = 120G
lxc.cgroup.memory.limit_in_bytes = 100G
```
### Disable unified and come back to hybrid 
#### On a host
```bash
# Check that you are using unified
ubuntu@server:~$ mount -l | grep cgroup
cgroup2 on /sys/fs/cgroup type cgroup2 (rw,nosuid,nodev,noexec,relatime,nsdelegate,memory_recursiveprot)

vim /etc/default/grub
#Append it (with a space) to `GRUB_CMDLINE_LINUX` in _/etc/default/grub_

update-grub

# Check the new kernel parameters
ubuntu@server:~$ grep systemd /boot/grub/grub.cfg
	linux	/vmlinuz-5.15.0-91-generic root=UUID=16571901-c6da-4bef-9201-5478098e78dd ro nomodeset iommu=pt console=tty0 console=ttyS0,115200n8 systemd.unified_cgroup_hierarchy=0
		linux	/vmlinuz-5.15.0-91-generic root=UUID=16571901-c6da-4bef-9201-5478098e78dd ro nomodeset iommu=pt console=tty0 console=ttyS0,115200n8 systemd.unified_cgroup_hierarchy=0
		linux	/vmlinuz-5.15.0-91-generic root=UUID=16571901-c6da-4bef-9201-5478098e78dd ro recovery nomodeset dis_ucode_ldr nomodeset iommu=pt console=tty0 console=ttyS0,115200n8 systemd.unified_cgroup_hierarchy=0

reboot

# Check that both cgroup v1 & v2 are now mounted
ubuntu@server:~$ mount -l | grep cgroup
tmpfs on /sys/fs/cgroup type tmpfs (ro,nosuid,nodev,noexec,size=4096k,nr_inodes=1024,mode=755,inode64)
cgroup2 on /sys/fs/cgroup/unified type cgroup2 (rw,nosuid,nodev,noexec,relatime,nsdelegate)
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,xattr,name=systemd)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,memory)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpu,cpuacct)
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,pids)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,perf_event)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,hugetlb)
cgroup on /sys/fs/cgroup/misc type cgroup (rw,nosuid,nodev,noexec,relatime,misc)
cgroup on /sys/fs/cgroup/rdma type cgroup (rw,nosuid,nodev,noexec,relatime,rdma)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,net_cls,net_prio)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,freezer)
```

#### On a LXC container
It is similar but simpler since systemd is not launched by grub but by LXC. Add the following line to the LXC configuration file `lxc.init.cmd = /sbin/init systemd.unified_cgroup_hierarchy=1`
## Sources
### cgroup v1
- [Introduction to Control Groups v1 by RedHat](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/resource_management_guide/ch01)
- [Other RedHat cgroup presentation](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/managing_monitoring_and_updating_the_kernel/setting-limits-for-applications_managing-monitoring-and-updating-the-kernel)
- [Arch Linux documentation](https://wiki.archlinux.org/title/cgroups)
- [Wikipedia page](https://en.wikipedia.org/wiki/Cgroups)
- [cgroup manpage](https://manpages.ubuntu.com/manpages/focal/en/man7/cgroups.7.html)
- https://www.cncf.io/blog/2023/06/30/embracing-cgroup-v2-best-practices-for-migrating-kubernetes-clusters-to-almalinux/
- [kernel documentation about cgroup v1](https://www.kernel.org/doc/Documentation/cgroup-v1/)(Presentation of the different subsystems)
### cgroup v2
- [Kernel documentation](https://www.kernel.org/doc/Documentation/cgroup-v2.txt)
- [Kubernetes documentation](https://kubernetes.io/docs/concepts/architecture/cgroups/)(Since Kubernetes is widely used and a big cgroup user there is a lot of litterature on this topic)
- [LXC and cgroup v2 (debian doc)](https://wiki.debian.org/LXC/CGroupV2)