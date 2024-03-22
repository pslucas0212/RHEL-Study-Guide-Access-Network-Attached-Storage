# RHEL Study Guide: Access Network-Attached Storage

[RHEL Study Guide - Table of Contents](https://github.com/pslucas0212/RHEL-Study-Guide) 

In this section we will configure the automounter with an indirect map, using exports from an NFSv4 server to support automounts for various users.

First we need to install the autofs service if it is not already installed.  Let's check for the installation.
```
# dnf list autofs
Last metadata expiration check: 0:17:17 ago on Fri 22 Mar 2024 04:05:35 PM EDT.
Available Packages
autofs.x86_64                         1:5.1.7-27.el9                         rhel-9.0-for-x86_64-baseos-rpms
```

Since autofs is not installed, we will go ahead and inistall it.
```
# dnf -y install autofs
Last metadata expiration check: 0:17:55 ago on Fri 22 Mar 2024 04:05:35 PM EDT.
Dependencies resolved.
============================================================================================================
 Package          Architecture     Version                  Repository                                 Size
============================================================================================================
Installing:
 autofs           x86_64           1:5.1.7-27.el9           rhel-9.0-for-x86_64-baseos-rpms           387 k
...
Complete!
```

Lets enable and start autofs.
```
# systemctl enable --now autofs
Created symlink /etc/systemd/system/multi-user.target.wants/autofs.service → /usr/lib/systemd/system/autofs.service.
```

Let's make sure we can access the /shares directory on serverb
```
# mount -t nfs serverb.lab.example.com:/shares /mnt
# ls /mnt
management  operation  production
# umount /mnt
```

Now we will configure an automounter indirect map servera with exports from serverb
```
# vi /etc/auto.master.d/shares.autofs
/remote /etc/auto.shares
# vi /etc/auto.shares
* -rw,sync,fstype=nfs4 serverb.lab.example.com:/shares/&
```

We will reboot servera to make sure the autofs service starts automatically
```
# systemctl reboot
$ ping -c3 servera
PING servera.lab.example.com (172.25.250.10) 56(84) bytes of data.
64 bytes from servera.lab.example.com (172.25.250.10): icmp_seq=1 ttl=64 time=0.843 ms
64 bytes from servera.lab.example.com (172.25.250.10): icmp_seq=2 ttl=64 time=0.281 ms
64 bytes from servera.lab.example.com (172.25.250.10): icmp_seq=3 ttl=64 time=0.329 ms

--- servera.lab.example.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 0.281/0.484/0.843/0.254 ms
```

ssh back to servera and test the configuration
```
# su - manager1
$ ls /remote/management
Welcome.txt
$ cat /remote/management/Welcome.txt 
###Welcome to Management Folder on SERVERB###
$ echo TEST1 > /remote/management/Test1.txt
$ cat /remote/management/Test1.txt 
TEST1
$ ls /remote/operation
ls: cannot open directory '/remote/operation': Permission denied
$ ls /remote/production
ls: cannot open directory '/remote/production': Permission denied
$ exit
logout
# su - dbuser1
$ ls /remote/production
Welcome.txt
$ cat /remote/production/Welcome.txt 
###Welcome to Production Folder on SERVERB###
echo TEST2 > /remote/production/Test2.txt
$ cat /remote/production/Test2.txt 
TEST2
$ ls /remote/operation/
ls: cannot open directory '/remote/operation/': Permission denied
$ ls /remote/management/
ls: cannot open directory '/remote/management/': Permission denied
$ exit
logout
# su - contractor1
$ ls /remote/operation
Welcome.txt
$ cat /remote/operation/Welcome.txt 
###Welcome to Operation Folder on SERVERB###
$ echo TEST3 > /remote/operation/Test3.txt
$ cat /remote/operation/Test3.txt 
TEST3
$ ls /remote/management
ls: cannot open directory '/remote/management': Permission denied
$ ls /remote/production/
ls: cannot open directory '/remote/production/': Permission denied
```

We are all done!










