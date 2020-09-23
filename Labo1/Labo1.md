## AIT Lab 01 - Linux Backup

**Author:** Müller Robin, Stéphane Teixeira Carvalho, Massaoudi Walid  
**Date:** 2020-09-25

### Task 1: Prepare the backup disk
***1.1 Which disks and which partitions on these disks are visible?***  
```bash
ubuntu@mobile:~$ ls /dev/hd*
ls: cannot access '/dev/hd*': No such file or directory
ubuntu@mobile:~$ ls /dev/sd*
/dev/sda  /dev/sda1
```
With the commands show above, we can see that one disk is available and it as one partition. We can deduce that because we have a disk named **sda** and it has only one number.

***1.2 Which new files appeared? These represent the disk and its partitions you just attached.***  

After the addition of a new disk, we can see that another disk is now available and his name is **sdb** as expected.
```bash
ubuntu@mobile:~$ ls /dev/sd*
/dev/sda  /dev/sda1  /dev/sdb
```

***1.3 Create a partition table on the disk and create two partitions of equal size using the parted tool***

Here are the different points of the manipulation :

****1.3.1 & 1.3.2 Use the parted command and display the existing partitions with the print command****
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
```

Has expected the disk is not recognised because it does not have a partition table.

****1.3.3 & 1.3.4 Use the mktable command to create a partition table and display the free space with the command print free****
```bash
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
After the mktable we can see, with the print free command, that the disk can now have partition because it shows the space available to create partitions.

****1.3.5 Creation of partitions****

Here is the result fot the **first partition**.
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
After the `print free` we can see that the partition has been set correctly. The partition has the fat32 file system, is a primary type and ends at half the free space.

Here is the result for the **second partition**.
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

As done with the previous parttion we used `print free` to check if the partition was correctly set. We can also see that the partition is correctly set.

****1.3.6 Quit parted and verify that there are now two special files in /dev that correspond to the two partitions.****

To verify if the two new special files were cretaed we used the `ls /dev/sd*`.
```bash
ubuntu@mobile:~$ ls /dev/sd*
/dev/sda  /dev/sda1  /dev/sdb  /dev/sdb1  /dev/sdb2
```

We can see that the two partitions are now created because the **/dev/sdb** disk has now two more files that corresponds to the partition created before(**/dev/sdb1** and **/dev/sdb2**).

***1.4 Format the two partitions using the mkfs command***

Here are the two commands done to completet this points.
```bash
ubuntu@mobile:~$ sudo mkfs.vfat /dev/sdb1
mkfs.fat 4.1 (2017-01-24)
```

```bash
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
```


***1.5 Create two empty directories in the /mnt directory as mount points, called backup1 and backup2. Mount the newly created file systems in these directories***

To mount the files systems in the correct directory we used the command `mount device mountpoint`

```bash
ubuntu@mobile:~$ sudo mount /dev/sdb1 /mnt/backup1
ubuntu@mobile:~$ sudo mount /dev/sdb2 /mnt/backup2
```

***1.6 How much free space is available on these filesystems? Use the df command to find out.***

```bash
ubuntu@mobile:~$ df -h /mnt/backup1
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdb1       8.0G  4.0K  8.0G   1% /mnt/backup1
ubuntu@mobile:~$ df -h /mnt/backup2
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdb2       3.9G   16M  3.7G   1% /mnt/backup2
```

***What does the -h option do?***

It can print size for humans in powers of 1024

<div style="page-break-after: always;"></div>

***Task 2: Perform backups using tar and zip***

* Do a backup of a user's home directory to the backup disk (VFAT partition). Create a compressed archive. Do the files in the archive have a relative path so that you can restore them later to any place?  
  We used the following command to backup the home directory :   
  `tar -cvpzf /mnt/backup1/backup.tar.gz ~`  
  -c is used to create a new archive file, -v to verbose, -p to preserve permissions, -z to compress with gzip and -f to specify the archive path.  
  The files in the archive contain a relative path. We confirmed it by displaying its files.


  `sudo zip -r /mnt/backup1/backup.zip ~`





* List the content of the archive.  
  `tar -ztf /mnt/backup1/backup.tar.gz`   
  -z allows us to filter the archive through gzip, -t to list the content of the archive and -f to specify the archive's path.

  `stephane@ubuntu:~$ unzip -l /mnt/backup1/backup.zip`
Archive:  /mnt/backup1/backup.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
        0  2020-09-23 06:25   home/stephane/
     3771  2020-09-23 06:08   home/stephane/.bashrc
      131  2020-09-23 06:13   home/stephane/.xinputrc
        0  2020-09-23 06:12   home/stephane/Desktop/
      807  2020-09-23 06:08   home/stephane/.profile
        0  2020-09-23 06:12   home/stephane/Pictures/
      954  2020-09-23 06:24   home/stephane/.ICEauthority
        0  2020-09-23 06:25   home/stephane/.sudo_as_admin_successful

* Do a restore of the archive to a different place, say /tmp.  
  `tar -zxvf /mnt/backup1/backup.tar.gz -C /tmp`  
  Same options as before, -x is used to extract the archive.

  `unzip /mnt/backup1/backup.zip -d /tmp`

* Do an incremental backup that saves only files that were modified after, say, September 23, 2016, 10:42:33. Do this only for tar, not for zip.
  `tar --listed-incremental=snapshot.file -cvzf backup.tar.gz -T <(find ~ -type f -newermt "2016-09-23 10:42:33")`

### Task 3

In this task you will examine how well the backup commands preserve file metadata. Consult the man pages and perform tests using tar and zip and examine whether you can restore:

the last modification time
the permissions
the owner
In the lab report describe the tests you did and their results.

stephane@ubuntu:~$ ls -l /tmp/home/stephane/
total 44
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Desktop
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Documents
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Downloads
-rw-r--r-- 1 stephane stephane 8980 sep 23 06:08 examples.desktop
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Music
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Pictures
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Public
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Templates
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Videos


stephane@ubuntu:~$ ls -l /tmp/home/stephane/
total 44
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Desktop
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Documents
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Downloads
-rw-r--r-- 1 stephane stephane 8980 sep 23 06:08 examples.desktop
drwxrwxrwx 2 root     stephane 4096 sep 23 07:15 Music
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Pictures
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Public
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Templates
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Videos

stephane@ubuntu:~$ unzip /mnt/backup1/backup.zip -d /tmp
Archive:  /mnt/backup1/backup.zip
replace /tmp/home/stephane/.bashrc? [y]es, [n]o, [A]ll, [N]one, [r]ename: A

stephane@ubuntu:~$ ls -l /tmp/home/stephane/
total 44
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Desktop
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Documents
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Downloads
-rw-r--r-- 1 stephane stephane 8980 sep 23 06:08 examples.desktop
drwxrwxrwx 2 root     stephane 4096 sep 23 07:15 Music
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Pictures
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Public
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Templates
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Videos
