## Lab 02 - File system snapshots


#### Pedagogical objectives

* Learn how to create snapshots of a file system

* Become familiar with rsync synchronization tool

* Perform backups to a remote system


### Task 1: Local sync

In this task you will create incremental backups of a home directory
by using file synchronization and file system snapshots. To store the
backup you will use the backup disk you partitioned in the previous
lab.

Put the full commands for each step and the output (shortened if too
long) into the lab report.

1.  Choose a home directory, preferably your own, for which to create
    a backup. A full uncompressed backup should fill the ext4
    partition of the backup disk to not more than 50%. If the home
    directory is too big, choose an appropriate sub-tree. To find out
    how much space is occupied by a directory tree you can use the
    command `du -sh <directory>`. The directory will be called the
    source directory.

    ```bash
    stephane@ubuntu:~$ du -sh ~
    7.3M	/home/stephane
    ```

2.  On the backup disk, on the ext4 partition, create a directory
    called `<username>_backup` that will contain the backup. Change
    the owner of this directory from root to the owner of the source
    directory so that (1) you do not need superuser rights to copy
    files into it and (2) the user can directly read files back from
    the directory without needing superuser rights. The directory will
    be called the backup directory.

    ```bash
    stephane@ubuntu:~$ sudo mkdir /mnt/backup1/stephane_backup
    stephane@ubuntu:~$ ls /mnt/backup1/
    stephane_backup
    stephane@ubuntu:~$ sudo chown stephane /mnt/backup1/stephane_backup
    stephane@ubuntu:~$ ls -l /mnt/backup1/
    total 4
    drwxr-xr-x 2 stephane root 4096 sep 30 05:55 stephane_backup
    ```

3.  Perform an initial copy of the source directory to the backup
    directory.

    In the backup directory you will create a series of
    sub-directories for the backups. Their name consists of a timestamp of the time the backup was made:

        * aeinstein_backup
            * 2017-09-25-093533
            * 2017-09-25-101142
            * 2017-09-25-104812

    Here the format for the timestamp is
    `<year>-<month>-<day>-<hour><minute><second>`.  Note that it is
    good practice to give the times in UTC to be independent of
    timezones.

    To perform the initial copy of the source directory into that
    directory you can use `rsync` with the `-a` and `-v` options:

            rsync -av <source_directory>/ 2017-09-25-093533

    * What do these options do?
      - `-a` : archive files and directory while synchronizing ( -a equal to following options -rlptgoD)
      - `-v` : will be able to make a verbose output
    * Specifically, which options are implied by the `-a` option and what do they do?
      - As shown above the option `-a` is a shortcut for the following options :
        - `-r` : sync files and directories recursively
        - `-l` : copy symlinks as symlinks during the sync
        - `-p` : preserve permissions
        - `-t` : preserve modification times
        - `-g` : preserve group
        - `-o` : preserve owner
        - `-D` : preserve device files and preserve special files
    * How can you use the `date` command to avoid typing the timestamp of the current time? How do you make `date` produce UTC time?
     - For that we can use the following command :
        - `date -u "+%Y-%m-%d-%H%M%S"`
          - the parameter `%Y` will show the year in the format 20XX
          - the parameter `%m` will show the month in digits
          - the parameter `$d` will show the days in digits
          - the parameter `%H` will show the hours
          - the parameter `%M` will show the minutes
          - the parameter `%S` will show the seconds
          - the option `-u` will show the time in UTC

          Here is the result of the command :
    ```bash
    stephane@ubuntu:~$ date -u "+%Y-%m-%d-%H%M%S"
    2020-09-30-131944
    ```

    After knowing all the commands to do the rsync we ran the following command :
    ```bash
    stephane@ubuntu:~$ rsync -av ~ /mnt/backup1/stephane_backup/$(date -u "+%Y-%m-%d-%H%M%S")
    sending incremental file list
    created directory /mnt/backup1/stephane_backup/2020-09-30-133536
    stephane/
    stephane/.ICEauthority
    stephane/.bash_history
    stephane/.bash_logout
    stephane/.bashrc
    stephane/.pam_environment
    stephane/.profile
    stephane/.sudo_as_admin_successful
    stephane/.xinputrc
    stephane/backup.tar.gz
    stephane/backupincr.tar.gz
    ...
    ```

    * How much disk space is used by the backup directory?

      ```bash
      stephane@ubuntu:~$ ls -l /mnt/backup1/stephane_backup
      total 4
      drwxr-xr-x 3 stephane stephane 4096 sep 30 15:35 2020-09-30-133536
      stephane@ubuntu:~$ du -sh /mnt/backup1/stephane_backup/
      7.3M	/mnt/backup1/stephane_backup/
      ```

      We can see that the backup directory as the same size as the source directory.

4.  Without having modified a file in the source directory do an
    incremental backup using hard links. As before the name of the
    destination directory is the current timestamp.

    In addition to the `rsync` options of the previous step, use the
    following options (use `man` to find out what they do):

    * `--delete` : delete extraneous files from destination dirs
    * `--link-dest` to do an incremental backup using hard links

    __Note:__ The `--link-dest` option takes a directory of a previous
    backup as parameter. If it is given as a __relative__ path, __rsync
    expects it to be relative not to the current directory, but the
    destination directory (the last parameter)__.

    Heres is the result of the rsync command :
    ```bash
    stephane@ubuntu:~$ rsync -av --delete --link-dest=/mnt/backup1/stephane_backup/2020-09-30-133536 ~ /mnt/backup1/stephane_backup/$(date -u "+%Y-%m-%d-%H%M%S")
    sending incremental file list
    created directory /mnt/backup1/stephane_backup/2020-09-30-135543
    stephane/.config/nautilus/
    stephane/.config/nautilus/desktop-metadata
    rsync: send_files failed to open "/home/stephane/.local/share/recently-used.xbel": Permission denied (13)
    stephane/.local/share/gvfs-metadata/
    stephane/.local/share/gvfs-metadata/root
    stephane/.local/share/gvfs-metadata/root-407fb817.log
    stephane/.local/share/zeitgeist/activity.sqlite-shm
    stephane/.local/share/zeitgeist/activity.sqlite-wal

    sent 336,245 bytes  received 370 bytes  673,230.00 bytes/sec
    total size is 6,293,293  speedup is 18.70
    ```

    How much disk space is used by the backup directory according to
    the `du` command? How much by the individual snapshot directories? How do
    you explain what `du` displays (if you had to write the `du` command, how would you count hard links)?

    ```bash
    stephane@ubuntu:~$ du -sh /mnt/backup1/stephane_backup/
    8.1M	/mnt/backup1/stephane_backup/
    stephane@ubuntu:~$ du -sh /mnt/backup1/stephane_backup/*
    7.3M	/mnt/backup1/stephane_backup/2020-09-30-133536
    744K	/mnt/backup1/stephane_backup/2020-09-30-135543
    ```

    With the `du` command we can see that all the files that were unchanged are hard linked to the first backup. This is due to the fact that we used `--linked-dest`

5.  Modify a file in the source directory and perform another
    incremental backup like in the previous step.

    For this manipulation we decide to change a file named `snapshot.file` in the home directory.

    When doing the rsync we can see that the file is added  :

    ```bash
    stephane@ubuntu:~$ rsync -av --delete --link-dest=/mnt/backup1/stephane_backup/2020-09-30-133536 ~ /mnt/backup1/stephane_backup/$(date -u "+%Y-%m-%d-%H%M%S")
    sending incremental file list
    created directory /mnt/backup1/stephane_backup/2020-09-30-140922
    stephane/
    stephane/snapshot.file
    IO error encountered -- skipping file deletion
    stephane/.config/nautilus/
    stephane/.config/nautilus/desktop-metadata
    stephane/.local/share/
    rsync: send_files failed to open "/home/stephane/.local/share/recently-used.xbel": Permission denied (13)
    stephane/.local/share/gvfs-metadata/
    stephane/.local/share/gvfs-metadata/root
    stephane/.local/share/gvfs-metadata/root-407fb817.log
    stephane/.local/share/nano/
    stephane/.local/share/zeitgeist/activity.sqlite-shm
    stephane/.local/share/zeitgeist/activity.sqlite-wal

    sent 336,366 bytes  received 404 bytes  673,540.00 bytes/sec
    total size is 6,293,297  speedup is 18.69
    ```

    Using the `stat` command examine the inodes of different versions
    of a file. Do this for a file that **has not** changed between
    backups and for a file that **has** changed. What do you see?

    Here is the result for a file that as changed :
    ```bash
    stephane@ubuntu:/mnt/backup1/stephane_backup$ stat 2020-09-30-140922/stephane/snapshot.file
      File: 2020-09-30-140922/stephane/snapshot.file
      Size: 40        	Blocks: 8          IO Block: 4096   regular file
    Device: 801h/2049d	Inode: 1208538     Links: 1
    Access: (0644/-rw-r--r--)  Uid: ( 1000/stephane)   Gid: ( 1000/stephane)
    Access: 2020-09-30 16:09:22.797117150 +0200
    Modify: 2020-09-30 16:07:33.112541821 +0200
    Change: 2020-09-30 16:09:22.797117150 +0200
     Birth: -
    stephane@ubuntu:/mnt/backup1/stephane_backup$ stat 2020-09-30-133536/stephane/snapshot.file
      File: 2020-09-30-133536/stephane/snapshot.file
      Size: 36        	Blocks: 8          IO Block: 4096   regular file
    Device: 801h/2049d	Inode: 1199058     Links: 2
    Access: (0644/-rw-r--r--)  Uid: ( 1000/stephane)   Gid: ( 1000/stephane)
    Access: 2020-09-30 15:35:36.387552314 +0200
    Modify: 2020-09-29 18:24:40.401061651 +0200
    Change: 2020-09-30 15:55:43.228826935 +0200
     Birth: -
    ```
    In this case we can see that the inode as changed and the hours of access too.

    In contrast, when a file is unchanged we can see that they use the same inode :
    ```bash
    stephane@ubuntu:/mnt/backup1/stephane_backup$ stat 2020-09-30-140922/stephane/myScan.xml
      File: 2020-09-30-140922/stephane/myScan.xml
      Size: 15845     	Blocks: 32         IO Block: 4096   regular file
    Device: 801h/2049d	Inode: 1198468     Links: 3
    Access: (0644/-rw-r--r--)  Uid: ( 1000/stephane)   Gid: ( 1000/stephane)
    Access: 2020-09-30 15:35:36.387552314 +0200
    Modify: 2020-09-25 13:07:25.000052231 +0200
    Change: 2020-09-30 16:09:22.789117109 +0200
     Birth: -
    stephane@ubuntu:/mnt/backup1/stephane_backup$ stat 2020-09-30-133536/stephane/myScan.xml
      File: 2020-09-30-133536/stephane/myScan.xml
      Size: 15845     	Blocks: 32         IO Block: 4096   regular file
    Device: 801h/2049d	Inode: 1198468     Links: 3
    Access: (0644/-rw-r--r--)  Uid: ( 1000/stephane)   Gid: ( 1000/stephane)
    Access: 2020-09-30 15:35:36.387552314 +0200
    Modify: 2020-09-25 13:07:25.000052231 +0200
    Change: 2020-09-30 16:09:22.789117109 +0200
     Birth: -

    ```

    How much disk space is used by the backup directory?
    ```bash
    stephane@ubuntu:/mnt/backup1/stephane_backup$ du -sh /mnt/backup1/stephane_backup/
    8.8M	/mnt/backup1/stephane_backup/
    ```

    As expected the directory only grew for about 700K due to the fact of the new backup.

6.  Delete the initial full backup. What happens to the files in the
    incremental backup that were hardlinked to the files of the full
    backup?

    ```bash
    stephane@ubuntu:/mnt/backup1/stephane_backup$ rm -rf 2020-09-30-133536
    stephane@ubuntu:/mnt/backup1/stephane_backup$ stat 2020-09-30-140922/stephane/myScan.xml
      File: 2020-09-30-140922/stephane/myScan.xml
      Size: 15845     	Blocks: 32         IO Block: 4096   regular file
    Device: 801h/2049d	Inode: 1198468     Links: 2
    Access: (0644/-rw-r--r--)  Uid: ( 1000/stephane)   Gid: ( 1000/stephane)
    Access: 2020-09-30 15:35:36.387552314 +0200
    Modify: 2020-09-25 13:07:25.000052231 +0200
    Change: 2020-09-30 16:26:34.774531979 +0200
     Birth: -
    ```

    We can see that the inode has not changed even after the deletion of the original backup. This is because some files were still referencing the original inode and so the system did not erase the inode.

### Task 2: Set up ssh for remote login

In this task you will configure SSH to easily log into a remote
virtual machine in a public cloud that will server as a backup
destination in the next task.

* The remote machine has URL address `ait.lan.iict.ch` (From Local Network or VPN).

* We have created an account for you on the machine. The account name
  is the part before the `@` in your email address. You must replace the `.` with `_`.

* The account is configured to
  accept login via SSH using a private key.

1. In your personal `.ssh` directory download the `key.sec` file below this document. Be sure to remove all permissions for `group` and
   `others` from this file.


2. Test logging into your account on the remote machine using SSH. Log
   out again.

3.  On your local machine configure an SSH shortcut to the account on the
    remote machine. Create the file `~/.ssh/config` if does not yet exist and
    add the following lines to it:

        # Cloud virtual machine for AIT lab
        Host cloudvm
            Hostname ait.lan.iict.ch
            IdentityFile ~/.ssh/key.sec
            User <my_user_name>

    Replace the username after `User` by
    your account name.

    Test this shortcut by typing `ssh cloudvm`. You should see the
    command line prompt of the remote machine.


### Task 3: Remote sync

In this task you will use the remote machine as backup destination.

Put the full commands for each step and the output (shortened if too
long) into the lab report.

1. Create a backup directory on the remote machine as described in
   Task 1 so that your user can read/write.

2. Repeat the full backup and the incremental backup of task 1, but
   with the backup going to the remote machine over SSH. In the
   `rsync` command you need to prefix the destination parameter with
   `cloudvm:` to tell `rsync` to use SSH to transfer the data to the
   remote machine.

3. Optional: Using a network monitoring tool on your local Linux
   machine like `bmon` observe how much network traffic `rsync` is
   causing.
