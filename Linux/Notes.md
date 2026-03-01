# Linux 

## Linux File System

Understanding the Linux directory structure is like learning the floor plan of a house. Unlike Windows, which uses drive letters (C:, D:), Linux uses a Unified Hierarchical Tree. Everything starts from the Root, represented by a single forward slash (/). 

Here is a breakdown of the most important directories and what they do:

>/bin and /usr/bin — Essential binaries (commands) like ls, cp, mv, grep. When you run any command, Linux looks here. In modern distros, /bin is often a symlink to /usr/bin.
<img width="1233" height="72" alt="image" src="https://github.com/user-attachments/assets/4d15a570-619a-4aac-ae2f-3226431e83c1" />

--

>/sbin and /usr/sbin — System binaries meant for the root user. Commands like fdisk, iptables, reboot live here.
<img width="554" height="68" alt="image" src="https://github.com/user-attachments/assets/1053fe04-1a8f-474a-8db0-84614e89721b" />

--

>/etc — This is where ALL configuration files live. Nginx config? /etc/nginx/. SSH config? /etc/ssh/. User info? /etc/passwd. If something is misbehaving in production, you’ll spend a lot of
time in /etc.
<img width="1414" height="80" alt="image" src="https://github.com/user-attachments/assets/5b063bae-b175-4342-87f2-1f177581bbaf" />

--

>/var — Variable data that changes constantly. Logs are at /var/log/. Databases often store data under /var/lib/. If your disk fills up in production, /var is usually the culprit.
<img width="1203" height="136" alt="image" src="https://github.com/user-attachments/assets/b8b5dd13-4bb8-48a2-a790-9d397fcf4605" />

--

>/home — Home directories for regular users. Each user gets /home/username.
<img width="457" height="32" alt="image" src="https://github.com/user-attachments/assets/bfbeb583-7262-4606-9908-7c512ec51ed9" />

--

>/root — Home directory specifically for the root user. Separate from /home on purpose.

>/tmp — Temporary files. Gets wiped on reboot. Applications use this for scratch space. Never store anything important here.
<img width="1440" height="50" alt="image" src="https://github.com/user-attachments/assets/5f8a8a24-f62c-4d75-a86d-010d87077986" />

--

>/proc — A virtual filesystem. It doesn’t exist on disk — the kernel generates it in memory. /proc/cpuinfo tells you about your CPU. /proc/meminfo tells you about RAM. /proc/<pid>/ shows info about a running process. Extremely useful for debugging.
<img width="1445" height="52" alt="image" src="https://github.com/user-attachments/assets/fbc5d1bd-df32-48f5-bde8-e50f64abd18a" />

The numbered directories (like 1, 10, 13, 25452) are Process IDs (PIDs).

    Every time a program runs, the Linux kernel assigns it a unique number.
    The /proc directory is a "virtual" filesystem. These folders don't exist on your hard drive; the kernel creates them in memory to show you real-time information about every running process.
<img width="1287" height="95" alt="image" src="https://github.com/user-attachments/assets/69a9558a-86bb-4566-bc82-b6c6af8d0e39" />

--

>/sys — Another virtual filesystem, similar to /proc, but more structured. Used to interact with the kernel and hardware.
<img width="730" height="37" alt="image" src="https://github.com/user-attachments/assets/b38fe471-f563-4599-bc01-0db5a0be1a4f" />

--

>/dev — Device files. Your hard disk is /dev/sda. A terminal is /dev/tty. /dev/null is the “black hole” — anything written to it disappears.

<img width="1475" height="51" alt="image" src="https://github.com/user-attachments/assets/4ac24f61-2e44-4053-8280-e8b6967b9a4c" />

Notice your disk drives are listed here. Depending on the cloud provider or hardware, they have different names:

    nvme0n1: This is your Non-Volatile Memory Express (NVMe) SSD.
    xvda: This is a "Xen Virtual Disk." Since you are on an ssm-user (likely an AWS EC2 instance), this is your virtual hard drive.
    
>/opt — Optional software. When you install third-party tools manually (not via package manager), they often go here.
<img width="413" height="36" alt="image" src="https://github.com/user-attachments/assets/0afb6453-f011-41c1-8872-2ff93094dcca" />

1. The Role of /usr/local/bin
In the Linux hierarchy, /usr/local is the "neighborhood" for software installed by the local administrator (you) that didn't come from the official OS repositories (like yum).

    /usr/bin: System-managed programs (installed via yum).
    /usr/local/bin: User-managed programs (installed via scripts or manual moves).
    Why here? Most Linux systems include /usr/local/bin in your $PATH by default. This means you can just type helm from anywhere, and it works immediately without you having to tell the system where it is.

2. Why not /opt?
If you look at your ls /opt output, you see aws. AWS usually puts its CLI or agent there because /opt is reserved for "Add-on" software packages that:

    Contain their own entire subdirectory structure (bin, lib, share, etc.).
    Are "self-contained" (like a giant folder for Google Chrome or an Oracle DB).
    Helm is just a single file (a binary). It doesn't need its own private folder and sub-folders, so it's cleaner to just drop that one file into /usr/local/bin.
   
--

>/usr — User-related programs and data. Think of it as a secondary hierarchy. /usr/bin, /usr/lib, /usr/share all live here.
<img width="699" height="38" alt="image" src="https://github.com/user-attachments/assets/bcdbb8a7-03e1-4c32-a8b6-63793b8ed77c" />

--

>/lib and /usr/lib — Shared libraries (.so files) that programs depend on. Like DLLs in Windows.
<img width="1184" height="51" alt="image" src="https://github.com/user-attachments/assets/e6732f12-ca11-46ff-9fdf-30c32c23a6aa" />

1. The Binary (/bin, /sbin, /usr/bin)
A binary is an executable file. It is a complete program that you can run directly from the terminal.

    Role: It performs a specific task from start to finish.
    Analogy: A Power Drill. You pick it up, pull the trigger (run it), and it does the job.
    Examples: ls, cd, java, helm, python.
    Where they live: Look in your $PATH folders (e.g., /usr/bin/ls).

2. The Library (/lib, /lib64, /usr/lib)
A library is a collection of pre-written code that binaries use to do their work. You cannot "run" a library by itself.

    Role: It provides shared functions (like "how to draw a window" or "how to connect to the internet") so that every single binary doesn't have to reinvent the wheel.
    Analogy: A Instruction Manual or a Battery. A battery is useless sitting on a shelf, but the Power Drill (Binary) needs it to function.
    Examples: libc.so (the standard C library), libssl.so (for encryption).
    Where they live: Usually in /lib or /usr/lib.
   
--

>/mnt and /media — Mount points. When you attach an external disk or NFS share, you mount it here.

>/boot — Bootloader files and the Linux kernel itself. Don’t touch this unless you know what you’re doing.
<img width="1278" height="69" alt="image" src="https://github.com/user-attachments/assets/d6737c68-09df-4060-bad3-b52c6adc5e7e" />

