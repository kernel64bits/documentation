Display partition information
```
ubuntu@server:~$ sudo parted /dev/nvme0n1

GNU Parted 3.2
Using /dev/nvme0n1
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) print                                                            
Model: INTEL SSDPEKKF512G8L (nvme)
Disk /dev/nvme0n1: 512GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system     Name                          Flags
 1      1049kB  274MB   273MB   fat32           EFI system partition          boot, hidden, esp
 2      274MB   290MB   16,8MB                  Microsoft reserved partition  msftres
 3      290MB   82,6GB  82,3GB  ntfs            Basic data partition          msftdata
 5      82,6GB  383GB   300GB   ext4
 6      383GB   387GB   4096MB  ext4
 8      387GB   486GB   99,8GB  ext4
 9      486GB   488GB   1081MB                  Encrypted
 7      488GB   511GB   23,5GB  linux-swap(v1)
 4      511GB   512GB   1049MB  ntfs            Basic data partition          hidden, diag
```
