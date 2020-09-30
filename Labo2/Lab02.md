## Lab 02 - File system snapshots


#### Pedagogical objectives

* Learn how to create snapshots of a file system

* Become familiar with rsync synchronization tool

* Perform backups to a remote system

###TASK 1: LOCAL SYNC
In this task you will create incremental backups of a home directory by using file synchronization and file system snapshots. To store the backup you will use the backup disk you partitioned in the previous lab.

Put the full commands for each step and the output (shortened if too long) into the lab report.

Choose a home directory, preferably your own, for which to create a backup. A full uncompressed backup should fill the ext4 partition of the backup disk to not more than 50%. If the home directory is too big, choose an appropriate sub-tree. To find out how much space is occupied by a directory tree you can use the command du -sh <directory>. The directory will be called the source directory.

On the backup disk, on the ext4 partition, create a directory called <username>_backup that will contain the backup. Change the owner of this directory from root to the owner of the source directory so that (1) you do not need superuser rights to copy files into it and (2) the user can directly read files back from the directory without needing superuser rights. The directory will be called the backup directory.

Perform an initial copy of the source directory to the backup directory.

In the backup directory you will create a series of sub-directories for the backups. Their name consists of a timestamp of the time the backup was made:

* aeinstein_backup
    * 2020-09-29-093533
    * 2020-09-29-101142
    * 2020-09-29-104812
Here the format for the timestamp is <year>-<month>-<day>-<hour><minute><second>. Note that it is good practice to give the times in UTC to be independent of timezones.

To perform the initial copy of the source directory into that directory you can use rsync with the -a and -v options:

    rsync -av <source_directory>/ 2020-09-29-093533
What do these options do?
Specifically, which options are implied by the -a option and what do they do?
How can you use the date command to avoid typing the timestamp of the current time? How do you make date produce UTC time?
How much disk space is used by the backup directory?
Without having modified a file in the source directory do an incremental backup using hard links. As before the name of the destination directory is the current timestamp.

In addition to the rsync options of the previous step, use the following options (use man to find out what they do):

--delete
--link-dest to do an incremental backup using hard links
Note: The --link-dest option takes a directory of a previous backup as parameter. If it is given as a relative path, rsync expects it to be relative not to the current directory, but the destination directory (the last parameter).

How much disk space is used by the backup directory according to the du command? How much by the individual snapshot directories? How do you explain what du displays (if you had to write the du command, how would you count hard links)?

Modify a file in the source directory and perform another incremental backup like in the previous step.

Using the stat command examine the inodes of different versions of a file. Do this for a file that has not changed between backups and for a file that has changed. What do you see?

How much disk space is used by the backup directory?

Delete the initial full backup. What happens to the files in the incremental backup that were hardlinked to the files of the full backup?

###TASK 2: ACCESS THE VM WITH SSH FOR REMOTE LOGIN
In this task you will use SSH to easily log into a remote virtual machine in a public cloud that will server as a backup destination in the next task.

The remote machine has URL address ec2-3-134-84-63.us-east-2.compute.amazonaws.com (From Local Network or VPN).

We have created an account for you on the machine. The account name is the part before the @ in your email address.

Your account is configured to accept login via SSH using the password : toortoor
```bash
osboxes@osboxes:~$  ssh walid.massaoudi@ec2-3-134-84-63.us-east-2.compute.amazonaws.com
```

###TASK 3: SSH AUTHENTICATION
SSH authentication by password ?!? Lol ! let's change it !

Don't forrget to put the configuration and the full commands and the output (shortened if too long) for each step into the lab report.

Create a .ssh folder at the root of your personal folder on the remote machine

```bash
walid.massaoudi@ip-172-31-35-19:~$ mkdir .ssh
walid.massaoudi@ip-172-31-35-19:~$ chmod go-rwx .ssh
walid.massaoudi@ip-172-31-35-19:~$ cd .ssh
walid.massaoudi@ip-172-31-35-19:~/.ssh$ touch authorized_keys
walid.massaoudi@ip-172-31-35-19:~/.ssh$ chmod go-rwx authorized_keys
```

and also we need :
```bash
osboxes@osboxes:~/.ssh$ touch config
osboxes@osboxes:~/.ssh$ chmod go-rwx config
```

Create a key pair on your local machine (or your VM) and then add it to the remote machine
```bash
osboxes@osboxes:~/.ssh$ ssh-keygen -C walid.massaoudi@heig-vd.ch
Generating public/private rsa key pair.
Enter file in which to save the key (/home/osboxes/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/osboxes/.ssh/id_rsa.
Your public key has been saved in /home/osboxes/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:LFr2DVL6RcQ7onWDboXbAZ1ysfr0dQ8VaN3JhcYAFvk walid.massaoudi@heig-vd.ch
The keys randomart image is:
+---[RSA 2048]----+
|         o**.o+o=|
|        oo*. o++o|
|        .*oo..  .|
|       +=oB E  . |
|      *+S*o+  o .|
|     +.=+*.. . o.|
|    .  .o o .   .|
|                 |
|                 |
+----[SHA256]-----+
```
add the public key  to the remote machine
```bash
walid.massaoudi@ip-172-31-35-19:~$ cat >> ~/.ssh/authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC4sVNfd77cZBnhOw8p5z5z2qjj0OlTO81Xcts7kv05tXeFVXJiOF74QQTjnj7YtCuJO7b5AQZOGGSib98fpGmnxAkQUtzF+tJC9g9ZveHunJuGJ8FUuexUxAK9Or+71PLiuSGkgRzLmDXo3UMVyb8HPOWP5lkc14CDyeUbksd5RdZVzxTN1fVAT5GevsA5dXQOgZkh3rLwLE3y2Ql/ZNmfIc9/As4C7wrZ/QDINMZ8kOiZN6nq4S22/qHg18WwQ5fdwE+R9GMMcpPUyxbqp6Bu3zJs7gLqb9ZVXezjrMFyJgf6uofql5ekcGKLHuvGPWOtZGIpM/ia9MgQYtXtluJR walid.massaoudi@heig-vd.ch

```
Configure a ssh shortcut on your local machine. It allows you to replace all parameters of the remote machine with a single shortcut when using ssh or rsync.
we simply add the following configuration  to  the  ~/.ssh/config file :

```bash
osboxes@osboxes:~/.ssh$ cat config

# Local Linux  virtual	machine	on VirtualBox
Host ubuntu
				HostName	localhost
				Port	2222
				User		osboxes.org
				IdentityFile	~/.ssh/id_rsa
# Shared host for AIT	labs
Host ait		       
        HostName	ec2-3-134-84-63.us-east-2.compute.amazonaws.com
				IdentityFile	~/.ssh/id_rsa
				User	walid.massaoudi

```

Test if it works !
```bash
osboxes@osboxes:~$ ssh ait
Welcome to Ubuntu 20.04.1 LTS (GNU/Linux 5.4.0-1024-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed Sep 30 14:15:34 UTC 2020

  System load:  0.0               Processes:             117
  Usage of /:   28.1% of 7.69GB   Users logged in:       2
  Memory usage: 27%               IPv4 address for eth0: 172.31.35.19
  Swap usage:   0%

 * Kubernetes 1.19 is out! Get it in one command with:

     sudo snap install microk8s --channel=1.19 --classic

   https://microk8s.io/ has docs and details.

29 updates can be installed immediately.
0 of these updates are security updates.
To see these additional updates run: apt list --upgradable


*** System restart required ***
Last login: Wed Sep 30 14:12:47 2020 from 193.134.219.71
```
You can find help in the document "ADS Lc01b Accès à distance avec SSH.pdf" p.17

###TASK 4: REMOTE SYNC
In this task you will use the remote machine as backup destination.

Put the full commands for each step and the output (shortened if too long) into the lab report.

Create a backup directory on the remote machine as described in Task 1 so that your user can read/write.

Repeat the full backup and the incremental backup of task 1, but with the backup going to the remote machine over SSH. In the rsync command you need to prefix the destination parameter with <task_3_shortcut>: to tell rsync to use SSH to transfer the data to the remote machine.

Optional: Using a network monitoring tool on your local Linux machine like bmon observe how much network traffic rsync is causing.
