## AIT Lab 01 - Linux Backup

**Author:** Müller Robin, Stéphane Teixeira Carvalho, Massaoudi Walid  
**Date:** 2020-09-30

### Task 1: Prepare the backup disk
***1.1 Which disks and which partitions on these disks are visible? Which partitions are mounted?***  
```bash
ubuntu@mobile:~$ ls /dev/hd*
ls: cannot access '/dev/hd*': No such file or directory
ubuntu@mobile:~$ ls /dev/sd*
/dev/sda  /dev/sda1
```
With the commands shown above, we can see that one disk is available and it has one partition. We can deduce that, because the disk is named **sda** and it has only one number. The number indicates the number of the partition.

```bash
stephane@ubuntu:~$ mount
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
udev on /dev type devtmpfs (rw,nosuid,relatime,size=1977244k,nr_inodes=494311,mode=755)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
tmpfs on /run type tmpfs (rw,nosuid,noexec,relatime,size=400228k,mode=755)
/dev/sda1 on / type ext4 (rw,relatime,errors=remount-ro)
securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev)
tmpfs on /run/lock type tmpfs (rw,nosuid,nodev,noexec,relatime,size=5120k)
...
```

With the mount command we can see all the mounted partitions. For instance, we can see that the partition **sda1** is mounted and has a filesystem of type ext4.

***1.2 Which new files appeared? These represent the disk and its partitions you just attached.***  

After the addition of a new disk, we can see that another disk is now available and his name is **sdb** as expected. When a new disk is added the letter of the disk will be incremented.
```bash
ubuntu@mobile:~$ ls /dev/sd*
/dev/sda  /dev/sda1  /dev/sdb
```

<div style="page-break-after: always;"></div>

***1.3 Create a partition table on the disk and create two partitions of equal size using the parted tool***

Here are the different points of the manipulation :

**1.3.1 & 1.3.2 Use the parted command and display the existing partitions with the print command**
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

As expected, the disk is not recognized because it does not have a partition table yet.

**1.3.3 & 1.3.4 Use the mktable command to create a partition table and display the free space with the command print free**
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
After the mktable we can see, with the print free command, that the disk can now have partitions because we initialized the disk with a partition table. We can now see the space available with the print free command.

<div style="page-break-after: always;"></div>

****1.3.5 Creation of partitions****

Here is the result for the **first partition**.
Here is a quick reminder of what the partition should be :

- be a primary partition
- have a file system type of fat32
- start at 0
- end at about half the free space.

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

Here is the result for the **second partition** .  
The specifications:

- be a primary partition
- have a file system type of ext4
- start at half the free space
- end at the free space.

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

As done with the previous partition, we used `print free` to check if the partition was correctly set. We can also see that it is indeed the case.

****1.3.6 Quit parted and verify that there are now two special files in /dev that correspond to the two partitions.****

To verify if the two new special files were created we used the command `ls /dev/sd*`.
```bash
ubuntu@mobile:~$ ls /dev/sd*
/dev/sda  /dev/sda1  /dev/sdb  /dev/sdb1  /dev/sdb2
```

We can see that the two partitions are now created because the **/dev/sdb** disk has now two more files that correspond to the partition created before(**/dev/sdb1** and **/dev/sdb2**).

***1.4 Format the two partitions using the mkfs command***

Below are the two commands used to complete this part.  
Firstly, the configuration of the vfat partition
```bash
ubuntu@mobile:~$ sudo mkfs.vfat /dev/sdb1
mkfs.fat 4.1 (2017-01-24)
```
Secondly, the configuration of the ext4 partition
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
 It can print the size of files for humans in powers of 1024 (i.e in KiB, MiB, GiB, ..).

<div style="page-break-after: always;"></div>

### Task 2: Perform backups using tar and zip

* Do a backup of a user's home directory to the backup disk (VFAT partition). Create a compressed archive. Do the files in the archive have a relative path so that you can restore them later to any place?

  We used the following command to backup the home directory :   
  `sudo tar -cvpzf /mnt/backup1/backup.tar.gz ~`  
  - `-c` is used to create a new archive file
  - `-v` to verbose
  - `-p` to preserve permissions
  - `-z` to compress with gzip
  - `-f` to specify the archive path.  

  The files in the archive contain a relative path. We confirmed it by displaying its files.

  For the zip file we used the following command :  
  `sudo zip -r /mnt/backup1/backup.zip ~`  
  - `-r` is used recursively add the files of the home directory.

  With this we have all the files from the home directory eaven the hidden ones. The zip contains also the relative path of the directory such as `home/user`

  In both commands we had to use **sudo** because if not we could not write in the `/mnt/backup1` folder. Due to his permissons.

* List the content of the archive.  

  `tar -ztf /mnt/backup1/backup.tar.gz`   
  - `-z` allows us to filter the archive through gzip
  - `-t` to list the content of the archive
  - `-f` to specify the archive's path.

  Here is a part of the command :
```bash
stephane@ubuntu:~$ tar -ztf /mnt/backup1/backup.tar.gz
home/stephane/
home/stephane/snap/
home/stephane/snap/nmap/
home/stephane/snap/nmap/current
home/stephane/snap/nmap/1356/
home/stephane/snap/nmap/common/
home/stephane/.bashrc
home/stephane/.zenmap/
home/stephane/.zenmap/zenmap_version
home/stephane/.zenmap/zenmap.conf
home/stephane/.zenmap/zenmap.db
home/stephane/.zenmap/target_list.txt
home/stephane/.zenmap/scan_profile.usp
home/stephane/.zenmap/recent_scans.txt
home/stephane/.xinputrc
home/stephane/Desktop/
home/stephane/.profile
...
```
<div style="page-break-after: always;"></div>

  `unzip -l /mnt/backup1/backup.zip`  
  `-l` allow us to unzip the file just to read the files in it.  
  Here is an example of the result for the command :
```bash
stephane@ubuntu:~$ unzip -l /mnt/backup1/backup.zip
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
        ...
```

* Do a restore of the archive to a different place, say `/tmp`.  

  `tar -zxvf /mnt/backup1/backup.tar.gz -C /tmp`  
  Same options as before, `-x` is used to extract the archive, `-C` to specify the output directory.

  `unzip /mnt/backup1/backup.zip -d /tmp`  
  We use the unzip command again but this time with the `-d` option to specify in which directory we would like to extract the file

* Do an incremental backup that saves only files that were modified after, say, September 23, 2016, 10:42:33. Do this only for tar, not for zip.

  `tar --listed-incremental=snapshot.file -cvzf backup.tar.gz -T <(find ~ -type f -newermt "2016-09-23 10:42:33")`

### Task 3 : Backup of file metadata

In this task you will examine how well the backup commands preserve file metadata.  
Consult the man pages and perform tests using `tar` and `zip` and examine whether you can restore:

- the last modification time
- the permissions
- the owner

Here is how we tested the different commands to see the result of a restoration :

First here is the result for the `tar` command.

```bash
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
```

In the beginning we can see that the files in `/tmp` contain exactly the same structure as the backup of the home directory.

Then we decided to change the permissions, the owner and added a new file to change the last modification time for the `Music` directory. Here is the final result.
```bash
stephane@ubuntu:~$ ls -l /tmp/home/stephane/
total 44
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Desktop
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Documents
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Downloads
-rw-r--r-- 1 stephane stephane 8980 sep 23 06:08 examples.desktop
drwxrwxrwx 2 root     stephane 4096 sep 23 07:27 Music
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Pictures
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Public
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Templates
drwxr-xr-x 2 stephane stephane 4096 sep 23 06:12 Videos
```

After that we ran the command `sudo tar -zxvf /mnt/backup1/backup.tar.gz -C /tmp` again and checked if the changes that we made were still there and we can see that the changes were dropped and the restore of the home directory is complete. We find the values from the backup again.

If the command is ran without the **sudo** command we found out that the metadata did not change in that case.

```bash
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
```

For the **zip** command we did the same procedure. Except that this time when we did the restore the directory kept the same permissons, owner and latest modification time. In this case the restore isn't "really" done. The metadata remains the same.

Here is the result of the unzip command and the result after the unzip :

```bash
stephane@ubuntu:~$ sudo unzip /mnt/backup1/backup.zip -d /tmp
Archive:  /mnt/backup1/backup.zip
replace /tmp/home/stephane/.bashrc? [y]es, [n]o, [A]ll, [N]one, [r]ename: A
```

```bash
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
```
<div style="page-break-after: always;"></div>

### Task 4 : Symbolic and hard links
In this task you will examine whether the backup commands preserve symbolic and hard links. Consult the man pages and perform tests using tar and zip.

First, we need to remember that a hard link is a direct reference to a file via its inode. However, Symbolic links are shortcuts that reference to a file instead of its inode value.

In the first step, we created a file called **hardlinkedFile** hard linked to the file **originalFileHard**. Then we created two others files with symbolic links.

Here is the creation of the files :

1. Creating a hard link between two files:
```bash
osboxes@osboxes:~/Documents$ ln originalFileHard hardlinkedFile
```
We can see that both files have now the same inode :
 ```bash
 osboxes@osboxes:~/Documents$ ls -i
262340 hardlinkedFile    262340 originalFileHard
```
2. Creating a symbolic link between two files :
```bash
osboxes@osboxes:~/Documents$ ln -s originalsym  symLinkedFile
```
 We can see that the symLinkedFile is linked to the originalsym file:

  ```bash
  osboxes@osboxes:~/Documents$ ls -la
  total 8
  drwxr-xr-x  2 osboxes osboxes 4096 Sep 23 11:55 .
  drwxr-xr-x 15 osboxes osboxes 4096 Sep 23 11:34 ..
  -rw-r--r--  2 osboxes osboxes    0 Sep 23 11:53 hardlinkedFile
  -rw-r--r--  2 osboxes osboxes    0 Sep 23 11:53 originalFileHard
  -rw-r--r--  1 osboxes osboxes    0 Sep 23 11:54 originalsym
  lrwxrwxrwx  1 osboxes osboxes   11 Sep 23 11:55 symLinkedFile -> originalsym
  ```

In the second step, we perform a backup as the previous task using zip.
```bash
osboxes@osboxes:/mnt/backup1$ sudo zip -r /mnt/backup1/backup.zip ~
```
Simply now we do a restore of the archive to **/tmp** using the unzip command :
```bash
osboxes@osboxes:/mnt/backup1$ sudo unzip /mnt/backup1/backup.zip  -d /tmp
```
<div style="page-break-after: always;"></div>

The result is predictable, by checking the content of our restored backup :
```bash
osboxes@osboxes:/tmp/home/osboxes/Documents$ ls -i
4588765 hardlinkedFile    4588766 originalsym
4588764 originalFileHard  4588763 symLinkedFile
```
```bash
osboxes@osboxes:/tmp/home/osboxes/Documents$ ls -la
total 8
drwxr-xr-x  2 root root 4096 Sep 23 11:55 .
drwxr-xr-x 15 root root 4096 Sep 23 11:34 ..
-rw-r--r--  1 root root    0 Sep 23 11:53 hardlinkedFile
-rw-r--r--  1 root root    0 Sep 23 11:53 originalFileHard
-rw-r--r--  1 root root    0 Sep 23 11:54 originalsym
-rw-r--r--  1 root root    0 Sep 23 11:54 symLinkedFile
```
At this point we can clearly see that the backup command zip did not preserve the hard link (the two files have not the same inode id ), neither the symbolic link.

Now, we will do the same backup-restore but using the tar command :

1. Performing a backup :
```bash
osboxes@osboxes:/$ sudo tar -cvpzf /mnt/backup1/backup.tar.gz ~
```
2. Restore the archive to /tmp:
```bash
osboxes@osboxes:/mnt/backup1$ sudo tar -zxvf /mnt/backup1/backup.tar.gz -C /tmp
```

As well as the previous task the backup command tar preserve both the hard link and the symblic one .this is the result :

```bash
osboxes@osboxes:/tmp/home/osboxes/Documents$ ls -la
total 8
drwxr-xr-x  2 osboxes osboxes 4096 Sep 23 11:55 .
drwxr-xr-x 15 osboxes osboxes 4096 Sep 23 11:34 ..
-rw-r--r--  2 osboxes osboxes    0 Sep 23 11:53 hardlinkedFile
-rw-r--r--  2 osboxes osboxes    0 Sep 23 11:53 originalFileHard
-rw-r--r--  1 osboxes osboxes    0 Sep 23 11:54 originalsym
lrwxrwxrwx  1 osboxes osboxes   11 Sep 23 11:55 symLinkedFile -> originalsym
osboxes@osboxes:/tmp/home/osboxes/Documents$ ls -i
4718659 hardlinkedFile    4718660 originalsym
4718659 originalFileHard  4718658 symLinkedFile
```
