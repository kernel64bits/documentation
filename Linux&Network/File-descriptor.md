

# File descriptor

# File descriptor

## List a process file descriptor
```bash
ubuntu@server:~$ pidof /usr/sbin/rsyslogd
12282
ubuntu@server:~$ sudo ls -la /proc/12282/fd | head
total 0
dr-x------. 2 root   root              0 May  2 09:27 .
dr-xr-xr-x. 9 syslog logreader-tenant  0 May  2 09:27 ..
lr-x------. 1 root   root             64 May  2 09:27 0 -> /dev/null
l-wx------. 1 root   root             64 May  2 09:27 1 -> /dev/null
lrwx------. 1 root   root             64 May  2 09:27 10 -> socket:[571140342]
```


Global file table
inode table
## Sources
https://www.cyberciti.biz/faq/linux-find-all-file-descriptors-used-by-a-process/
