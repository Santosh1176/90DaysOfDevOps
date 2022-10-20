# Linux Filesystems

Linux supports several types of filesystems to manage files and folders. Each filesystem implements the virtual directory structure on storage
devices using slightly different features.


In Linux everything is a file

To see how the Linux filesystem is laid out try this with `tree -L 1 /` command:

```bash
 santosh@~*$:tree -L 1 /
/
├── bin -> usr/bin
├── boot
├── cdrom
├── dev
├── etc
├── home
├── lib -> usr/lib
├── lib32 -> usr/lib32
├── lib64 -> usr/lib64
├── libx32 -> usr/libx32
├── lost+found
├── media
├── mnt
├── opt
├── proc
├── root
├── run
├── sbin -> usr/sbin
├── snap
├── srv
├── swapfile
├── sys
├── tmp
├── usr
└── var
```

The above commands produces a high level (ist Level) tree structure of the `root` directory of the system.

Lets discuss the directories listed above:

- `/bin`
/bin is the directory that contains binaries, that is, some of the applications and programs you can run. Almost all basic Linux commands can be found here such ls, cat, touch, pwd, rm, echo, … The binaries on this directory must be available in order to attain minimal functionality for the purposes of booting and repairing a system.

- `/boot`
Contains the Linux kernel, initial RAM disk image (for drivers needed at boot time), and the boot loader. Interesting files include /boot/
grub/grub.conf, or menu.lst, which is used to configure the boot loader, and /boot/vmlinuz (or something similar), the Linux kernel.


- `dev`
This is a special directory that contains device nodes. “*Everything is a file*” also applies to devices. Here is where the kernel maintains a list
of all the devices it understands.

- `/etc`
The /etc *etc stands for “Everything to configure,”* directory contains all the system-wide configuration files. It also contains a collection of shell scripts that start each of the system services at boot time. Everything in this directory should be readable text. While everything in /etc is interesting, here are some all-time
favorites: /etc/crontab, a file that defines when automated jobs will run; /etc/fstab, a table of storage devices and their associated mount
points; and /etc/passwd, a list of the user accounts.

- `/home`
In normal configurations, each user is given a directory in /home. Ordinary users can write files only in their home directories. This limitation protects the system from errant user activity. 

- `/lib`
Contains shared *library* files used by the core system programs.These are similar to dynamic link libraries (DLLs) in Windows. There are more `lib` directories scattered around the file system, but this one, the one hanging directly off of `/` is special in that, among other things, it contains the all-important kernel modules. 

- `/lost+found`
Each formatted partition or device using a Linux file system, such as ext3, will have this directory. It is used in the case of a partial recovery from a file system corruption event. Unless something really bad has happened to your system, this directory will remain empty.

- `/media`
On modern Linux systems, the /media directory will contain the mount points for removable media such as USB drives, CD-ROMs, and so on, that are mounted automatically at insertion.

- `/mnt`
On older Linux systems, the /mnt directory contains mount points for removable devices that have been mounted manually.  It is not used very often nowadays.

- `/opt`
The /opt directory is used to install *“optional”* software. This is mainly used to hold commercial software products that might be installed on the system. However, another place where applications and libraries end up in is `/usr/local`, When software gets installed here, there will also be `/usr/local/bin` and `/usr/local/lib` directories. What determines which software goes where is how the developers have configured the files that control the compilation and installation process.
*When we install `weave` or `flannel` CNI pluggins. for POD networking, those binaries will be stored in `/opt`.*

- `/proc`
Its like `/dev` is a virtual directory. It contains information about your computer, such as information about your CPU and the kernel your Linux system is running. As with `/dev`, the files and directories are generated when your computer starts, or on the fly, as your system is running and things change.

- `/sbin`
This directory contains *“system binaries"*. These are programs that perform vital system tasks that are generally reserved for the superuser / Root User

- `/tmp`
The `/tmp` directory is intended for the storage of temporary, transient files created by various programs. *Some configurations cause this directory to be emptied each time the system is rebooted*.

- `usr`
The `/usr` directory tree is likely the largest one on a Linux system. It contains all the programs and support files used by regular users. You will also find `bin`, `sbin` and `lib` directories in `/usr`. `/usr/bin` on the other hand would contain stuff the users would install and run to use the system as a work station and `/usr/lib` contains the shared *libraries* for the programs in `/usr/bin`.  The `/usr/local` is a tree where programs that are not included with your distribution but are intended for system-wide use are installed and `/usr/sbin` contains more system administration programs. The `/var` irectory tree is where data that is likely to change
is stored. Various databases, spool files, user mail, and so forth, are located here. For maintaining system Logs Linux maintaines `/var/log` directory which contains log files, records of various system activity. These are important and should be monitored from time to time. The most useful ones are `/var/log/messages` and `/var/log/syslog`.

As said earlier, this might be overwhelming at first look and though many Linux distributions may have some varying flavour of a Filesystem but they will be similar to the one outlined above. The best way to know the filesystem is to explore it. Jus check what are the contents of the directory by passing `ls -l <directory Path>`. You cannot damage your filesystem just by looking at it, so move from one directory to another and take a look around. Soon you’ll discover that the Linux filesystem and how it is laid out really makes a lot of sense, and you will intuitively know where to find apps, documentation, and other resources. 

# Resources:
- [Classic SysAdmin: The Linux Filesystem Explained](https://www.linuxfoundation.org/blog/blog/classic-sysadmin-the-linux-filesystem-explained)
- [General overview of the Linux file system](https://tldp.org/LDP/intro-linux/html/sect_03_01.html)