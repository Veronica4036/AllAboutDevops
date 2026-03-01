## Linux 

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

--

>/usr — User-related programs and data. Think of it as a secondary hierarchy. /usr/bin, /usr/lib, /usr/share all live here.
<img width="699" height="38" alt="image" src="https://github.com/user-attachments/assets/bcdbb8a7-03e1-4c32-a8b6-63793b8ed77c" />

--

>/lib and /usr/lib — Shared libraries (.so files) that programs depend on. Like DLLs in Windows.
<img width="1184" height="51" alt="image" src="https://github.com/user-attachments/assets/e6732f12-ca11-46ff-9fdf-30c32c23a6aa" />

--

>/mnt and /media — Mount points. When you attach an external disk or NFS share, you mount it here.

>/boot — Bootloader files and the Linux kernel itself. Don’t touch this unless you know what you’re doing.
<img width="1278" height="69" alt="image" src="https://github.com/user-attachments/assets/d6737c68-09df-4060-bad3-b52c6adc5e7e" />
