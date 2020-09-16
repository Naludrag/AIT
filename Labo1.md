## ADS Lab 09 - PowerShell

**Author:** Müller Robin, Stéphane Teixeira Carvalho  
**Date:** 2020-06-25

### Task 1: Prepare the backup disk
***1.1 Which disks and which partitions on these disks are visible?***  
```bash
ubuntu@mobile:~$ ls /dev/hd*
ls: cannot access '/dev/hd*': No such file or directory
ubuntu@mobile:~$ ls /dev/sd*
/dev/sda  /dev/sda1
```
***1.2 Which new files appeared? These represent the disk and its partitions you just attached.***  

```bash
ubuntu@mobile:~$ ls /dev/sd*
/dev/sda  /dev/sda1  /dev/sdb
```

***1.3 Create a partition table on the disk and create two partitions of equal size using the parted tool***

```bash
ubuntu@mobile:~$ sudo parted /dev/sdb
[sudo] password for ubuntu:
GNU Parted 3.2
Using /dev/sdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) print                                                            
Error: /dev/sdb: unrecognised disk label
Model: VMware, VMware Virtual S (scsi)                                    
Disk /dev/sdb: 17.2GB
Sector size (logical/physical): 512B/512B
Partition Table: unknown
Disk Flags:
(parted) mktable                                                          
New disk label type? msdos                                                
(parted) print free                                                       
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdb: 17.2GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type  File system  Flags
        32.3kB  17.2GB  17.2GB        Free Space

(parted)
```

```bash
(parted) mkpart                                                     
Partition type?  primary/extended? primary                                
File system type?  [ext2]? fat32                                          
Start? 0                                                                  
End? 50%                                                                  
Warning: The resulting partition is not properly aligned for best performance.
Ignore/Cancel? Ignore                                                     
(parted) print free
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdb: 17.2GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
        0.00B   511B    512B             Free Space
 1      512B    8590MB  8590MB  primary  fat32        lba
        8590MB  17.2GB  8590MB           Free Space

(parted)
```

```bash
(parted) mkpart
Partition type?  primary/extended? primary                                
File system type?  [ext2]? ext4                                           
Start? 75%                                                                
End? 100%                                                                 
(parted) print free
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdb: 17.2GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
        0.00B   511B    512B             Free Space
 1      512B    8590MB  8590MB  primary  fat32        lba
        8590MB  12.9GB  4295MB           Free Space
 2      12.9GB  17.2GB  4295MB  primary  ext4         lba

(parted)
```

***1.4 Format the two partitions using the mkfs command***

```bash
ubuntu@mobile:~$ sudo mkfs.vfat /dev/sdb1
mkfs.fat 4.1 (2017-01-24)
ubuntu@mobile:~$ sudo mkfs.ext4 /dev/sdb2
mke2fs 1.44.1 (24-Mar-2018)
Creating filesystem with 1048576 4k blocks and 262144 inodes
Filesystem UUID: 17c8bf37-9b9c-46d6-b4f2-d947eba56453
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

ubuntu@mobile:~$
```


***1.5 Create two empty directories in the /mnt directory as mount points, called backup1 and backup2. Mount the newly created file systems in these directories***

```bash
ubuntu@mobile:~$ sudo mount /dev/sdb1 /mnt/backup1
ubuntu@mobile:~$ sudo mount /dev/sdb2 /mnt/backup2
ubuntu@mobile:~$ ls /dev/sd*                                     
/dev/sda  /dev/sda1  /dev/sdb  /dev/sdb1  /dev/sdb2
```

***1.6 How much free space is available on these filesystems? Use the df command to find out. What does the -h option do?***

```bash
ubuntu@mobile:~$ df -h /mnt/backup1
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdb1       8.0G  4.0K  8.0G   1% /mnt/backup1
ubuntu@mobile:~$ df -h /mnt/backup2
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdb2       3.9G   16M  3.7G   1% /mnt/backup2

What does the -h option do?
It can print size for humans in powers of 1024

```

<div style="page-break-after: always;"></div>
