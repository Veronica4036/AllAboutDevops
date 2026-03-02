

# �� Section 1: Linux Fundamentals — DevOps Training Handbook

> **Mentor's Note:** This section builds the foundation everything else sits on. A DevOps engineer is expected to not just *use* these concepts but *explain* them under pressure, debug them at 2 AM, and make architectural decisions based on them. Every topic here has caused a production outage somewhere — learn it like your on-call rotation depends on it.

---

## 1.1 Linux Architecture & Boot Process

### Concept Deep Dive

Linux is a **monolithic kernel** — the kernel runs in a single memory space and directly manages hardware, memory, processes, and I/O. Unlike microkernels, device drivers run inside kernel space, which makes it fast but means a bad driver can crash the whole system.

**The three layers of a running Linux system:**

```
┌─────────────────────────────────────┐
│           User Space                │
│   (your apps, shell, libraries)     │
├─────────────────────────────────────┤
│      System Call Interface          │
│  (glibc, syscalls: read/write/fork) │
├─────────────────────────────────────┤
│           Kernel Space              │
│  (process mgmt, memory, VFS, net)   │
├─────────────────────────────────────┤
│             Hardware                │
│      (CPU, RAM, NIC, Disk)          │
└─────────────────────────────────────┘
```

User space processes **cannot** directly access hardware — they must ask the kernel via **system calls**. This boundary is enforced by CPU privilege rings (Ring 0 = kernel, Ring 3 = user). This is why a crashing application doesn't take down the OS — it's isolated in user space.

---

### The Boot Sequence — Step by Step

```
Power ON
   │
   ▼
BIOS / UEFI
   │  POST (Power-On Self Test), finds bootable device
   ▼
Bootloader (GRUB2)
   │  Loads kernel image (vmlinuz) + initramfs into RAM
   │  /boot/grub2/grub.cfg defines boot entries
   ▼
Kernel Initialization
   │  Decompresses itself, initializes CPU, memory, drivers
   │  Mounts initramfs (temporary root filesystem in RAM)
   │  Finds and mounts the real root filesystem
   ▼
PID 1 — systemd (or init)
   │  First userspace process, parent of everything
   │  Reads unit files, starts services in dependency order
   ▼
Default Target (multi-user.target or graphical.target)
   │  All services started, system ready
   ▼
Login prompt
```

**Why initramfs matters in production:**
`initramfs` is a minimal root filesystem loaded into RAM during boot. It contains drivers needed to mount the real root filesystem (e.g., LVM drivers, LUKS encryption drivers, NVMe drivers for Nitro-based EC2 instances). If initramfs is missing a driver, the system won't boot. This is why after installing a new kernel or changing storage configuration, you must rebuild it:

```bash
# RHEL / Amazon Linux
dracut -f /boot/initramfs-$(uname -r).img $(uname -r)

# Ubuntu / Debian
update-initramfs -u -k $(uname -r)
```

---

### Runlevels vs systemd Targets

| SysV Runlevel | systemd Target | Purpose |
|---|---|---|
| 0 | `poweroff.target` | Shutdown |
| 1 | `rescue.target` | Single-user / recovery |
| 3 | `multi-user.target` | Multi-user, no GUI (servers) |
| 5 | `graphical.target` | Multi-user with GUI |
| 6 | `reboot.target` | Reboot |

```bash
# Check current target
systemctl get-default

# Change default target
systemctl set-default multi-user.target

# Switch target immediately
systemctl isolate rescue.target

# Boot time analysis
systemd-analyze
systemd-analyze blame              # which service took longest
systemd-analyze critical-chain     # critical path to boot
```

---

### Real-World Production Scenarios

**Scenario 1: Server boots into emergency mode after a bad `/etc/fstab` entry**

A team adds an NFS mount to `/etc/fstab` without the `nofail` option. The NFS server is unreachable during boot. The system drops into emergency mode and the application is down.

```bash
# During emergency mode, root filesystem is read-only
# Remount it read-write to fix
mount -o remount,rw /

# Edit the bad fstab entry
vi /etc/fstab

# The fix: always add nofail and _netdev for network mounts
# 10.0.0.5:/data /mnt/nfs nfs defaults,nofail,_netdev 0 0
# _netdev = wait for network before mounting
# nofail  = don't fail boot if mount fails

systemctl reboot
```

**Scenario 2: Kernel panic after a botched kernel update on a production EC2 instance**

```bash
# Prevention: always keep the previous kernel in GRUB
grubby --info=ALL

# Set previous kernel as default before updating
grubby --set-default /boot/vmlinuz-<previous-version>

# On AWS: use EC2 Serial Console or create an AMI before kernel updates
# Recovery: stop instance → detach root EBS → attach to rescue instance
# → fix grub.cfg → reattach → start

# Check running kernel version
uname -r
```

**Scenario 3: Boot is slow — a service is timing out during startup**

```bash
# Identify the slow service
systemd-analyze blame | head -20

# Check the critical dependency chain
systemd-analyze critical-chain

# Common culprits:
# - cloud-init waiting for instance metadata
# - A service with 90s timeout waiting for a dependency that never comes up
# - Network-dependent service starting before network is ready
```

---

### Interview Questions & Answers

**Q1: A production EC2 instance running a critical payment service fails to boot after a routine OS patch. The instance shows "kernel panic — not syncing: VFS: Unable to mount root fs." Walk me through your recovery process.**

> First, I'd stop the instance (not terminate) and snapshot the root EBS volume as a safety net. I'd detach the root volume and attach it to a healthy rescue EC2 instance in the same AZ. After mounting it (`mount /dev/xvdf1 /mnt/rescue`) and chrooting (`chroot /mnt/rescue`), I'd investigate. The error "Unable to mount root fs" typically means initramfs is missing a storage driver, or the root device name changed — common when migrating to a Nitro-based instance type where `/dev/xvda` becomes `/dev/nvme0n1`. I'd check `/boot/grub2/grub.cfg` for the correct root device, rebuild initramfs with `dracut -f`, then detach, reattach to the original instance, and boot. If it's an instance type change, I'd also verify ENA and NVMe drivers are present in initramfs before the migration.

---

**Q2: During a post-incident review, you find that a service consistently takes 4 minutes to start during boot, delaying the entire application stack. How do you diagnose and fix this without modifying the application code?**

> I'd run `systemd-analyze critical-chain <service>` to map the dependency chain and find where time is lost, then `systemd-analyze blame` to confirm the service is the bottleneck. Common causes: the unit has `After=network-online.target` but full network readiness takes time, or it's polling a DNS name that resolves slowly at boot. I'd audit the unit file for unnecessary `After=` dependencies — replacing `network-online.target` with `network.target` if full readiness isn't required. I'd add `TimeoutStartSec=30` to fail fast rather than hang indefinitely. If the service does health checks against an external endpoint at startup, I'd add a `ExecStartPre=` pre-start script with retry logic so the dependency wait is handled gracefully rather than blocking the entire boot sequence.

---

**Q3: You're onboarding a new application to a fleet of 500 EC2 instances. The app requires a specific kernel parameter set at boot. How do you implement this reliably across the fleet, and how do you verify it persisted after a reboot?**

> Kernel parameters are set persistently via drop-in files in `/etc/sysctl.d/`. I'd create `/etc/sysctl.d/99-myapp.conf` with the required parameters (e.g., `net.core.somaxconn = 65535`). For the fleet, I'd use AWS Systems Manager Run Command to write the file and apply it immediately with `sysctl --system`. I'd bake this into the AMI via a Packer pipeline so all new instances have it from launch. For verification: an SSM Run Command document that runs `sysctl <parameter>` across all instances and reports results to S3. For ongoing compliance, I'd add an AWS Config custom rule that checks the parameter value and flags non-compliant instances — this catches drift from manual changes or failed bootstraps.

---

**Q4: What is the role of initramfs and when would you need to rebuild it in a production environment?**

> `initramfs` is a compressed archive the kernel extracts into a temporary RAM-based root filesystem during early boot. It contains the minimal drivers and tools needed to mount the real root filesystem — LVM drivers if root is on LVM, LUKS tools if the disk is encrypted, NVMe drivers for Nitro-based EC2 instances. You need to rebuild it when: changing the root filesystem type, adding disk encryption, migrating an EC2 instance to a Nitro-based instance type, or after certain kernel module changes. Failing to rebuild after a storage change is the most common cause of "unable to mount root fs" panics in production. On RHEL/Amazon Linux: `dracut -f`. On Ubuntu: `update-initramfs -u`.

---

## 1.2 Process Management

### Concept Deep Dive

Every running program on Linux is a **process** — an isolated execution context with its own virtual memory space, file descriptors, credentials, and scheduling state. The kernel tracks every process in a linked list of `task_struct` structures (the process table).

**The fork-exec model — how every process is born:**

```
Shell (bash)
    │
    │  fork()
    ├──────────────────────────────────────────────────────────────────────┐
    │                                                                      │
    ▼                                                                      ▼
Parent (bash continues)                                     Child (copy of bash)
                                                                           │
                                                              exec("/usr/bin/nginx")
                                                                           │
                                                                           ▼
                                                                  nginx process
```

`fork()` creates an exact copy of the calling process using **Copy-on-Write (CoW)** — the child shares the parent's memory pages until either writes, at which point a private copy is made. This makes `fork()` cheap even for large processes.

---

### Process States

```
         fork()
           │
           ▼
        RUNNABLE (R) ◄──────────────────────────────────────────────────────┐
           │                                                                 │
    scheduled by kernel                                              I/O completes
           │                                                                 │
           ▼                                                                 │
        RUNNING (R)                                                          │
           │                                                                 │
    waits for I/O ──────────────────────────────────────────────────► SLEEPING (S)
           │                                                                 │
    waits for disk ─────────────────────────────────────────────────► UNINTERRUPTIBLE (D)
           │
    exit() called ──────────────────────────────────────────────────► ZOMBIE (Z)
                                                                      (until parent calls wait())
```

---

### Signals Reference

| Signal | Number | Default Action | Common Use |
|---|---|---|---|
| SIGHUP | 1 | Terminate | Reload config (nginx, sshd) |
| SIGINT | 2 | Terminate | Ctrl+C |
| SIGQUIT | 3 | Core dump | Ctrl+\\, Java thread dump |
| SIGKILL | 9 | Terminate (uncatchable) | Force kill |
| SIGTERM | 15 | Terminate | Graceful shutdown |
| SIGCHLD | 17 | Ignore | Child process changed state |
| SIGSTOP | 19 | Stop (uncatchable) | Pause process |
| SIGCONT | 18 | Continue | Resume stopped process |

---

### Process Priority and Scheduling

Linux uses the **Completely Fair Scheduler (CFS)**. Each process has a `nice` value (-20 to +19) that influences its CPU time share. Lower nice = higher priority. Only root can set negative nice values.

```bash
# Start with lower priority
nice -n 10 ./heavy-batch-job.sh

# Change priority of a running process
renice -n 15 -p <PID>

# Real-time priority (for latency-sensitive workloads)
chrt -f 50 ./realtime-app    # FIFO scheduling, priority 50
```

---

### cgroups — Resource Control

cgroups (control groups) limit and account for resource usage of process groups. This is the foundation of container resource limits. Every systemd service runs in its own cgroup.

```bash
# See cgroup hierarchy
systemd-cgls

# Check memory usage by cgroup
cat /sys/fs/cgroup/memory/system.slice/nginx.service/memory.usage_in_bytes

# Check CPU usage
cat /sys/fs/cgroup/cpu/system.slice/nginx.service/cpuacct.usage
```

---

### Real-World Production Scenarios

**Scenario 1: A Java microservice on EKS is consuming 100% CPU on one node, causing other pods to be throttled**

```bash
# On the node (via kubectl debug or SSH)
top -b -n 1 | head -20
ps aux --sort=-%cpu | head -10

# Get the PID of the Java process
pgrep -a java

# Get a thread dump without restarting (SIGQUIT to Java = thread dump to stdout)
kill -3 <PID>
kubectl logs <pod-name> --tail=100

# Use jstack if JDK is available in the container
kubectl exec -it <pod> -- jstack <PID>

# Look for: threads in RUNNABLE state doing tight loops
# Common causes: infinite loop, regex catastrophic backtracking,
# GC thrashing (heap too small), connection pool exhaustion causing spin-wait

# Check if CPU limits are set
kubectl describe pod <pod> | grep -A5 "Limits\|Requests"
# If no limits: add them
# If limits too low: CPU throttling shows as high %sys time
```

**Scenario 2: Hundreds of zombie processes accumulating on a container host, eventually exhausting the PID namespace**

```bash
# Identify zombies
ps aux | awk '$8 == "Z" {print $0}'

# Count them
ps aux | awk '$8 == "Z"' | wc -l

# Find the parent (the broken process)
ps -o pid,ppid,stat,cmd aux | awk '$3 ~ /Z/ {print $2}' | sort -u

# Option 1: Signal parent to reap children
kill -CHLD <PPID>

# Option 2: Kill the parent (zombies reparent to PID 1 which reaps them)
kill <PPID>

# Root cause in containers: if PID 1 is your app binary (not tini/dumb-init),
# it won't reap zombies. Fix in Dockerfile:
# ENTRYPOINT ["/tini", "--", "/app/start.sh"]
```

**Scenario 3: A deployment pipeline spawns background processes and hangs indefinitely waiting for them**

```bash
#!/bin/bash
set -e

# Start background jobs and capture PIDs
./worker1.sh &
PID1=$!
./worker2.sh &
PID2=$!

# Wait for all and capture exit codes
wait $PID1 || { echo "worker1 failed"; exit 1; }
wait $PID2 || { echo "worker2 failed"; exit 1; }

# Use timeout to prevent indefinite hangs
timeout 300 ./long-running-task.sh || { echo "Task timed out"; exit 1; }

# Ensure cleanup on exit
trap 'kill $(jobs -p) 2>/dev/null' EXIT
```

---

### Interview Questions & Answers

**Q1: Your monitoring shows a Node.js application's event loop lag spiking to 5 seconds every 30 minutes, causing request timeouts. The CPU usage looks normal. How do you identify the root cause using Linux process tools?**

> Node.js is single-threaded, so event loop lag means something is blocking the loop — not necessarily CPU-bound work. I'd start with `strace -p <PID> -c` to get a syscall frequency summary over 60 seconds, looking for blocking syscalls like `read`, `write`, or `futex` with high accumulated time. Then `lsof -p <PID>` to check for file descriptor leaks — if the process has thousands of open FDs, each operation gets slower. I'd check `/proc/<PID>/status` for `VmSwap` — if the process is swapping, that explains the periodic lag. The 30-minute interval strongly suggests a scheduled task; I'd correlate with application logs to find what runs on that cadence (GC, cache flush, cron-triggered API call). For a definitive answer, I'd use `perf record -p <PID> -g -- sleep 30` during the spike and `perf report` to generate a flame graph of where time is spent.

---

**Q2: After a deployment, you notice the application's PID 1 in the container is your app binary, not an init process. Why is this a problem and what are the production implications?**

> PID 1 has special responsibilities in Linux: it must reap orphaned processes by calling `wait()`, and it receives signals differently — SIGTERM sent to a container goes directly to PID 1. Most application binaries (Node.js, Java, Python) don't implement zombie reaping or proper signal forwarding. The production implications are: zombie processes accumulate over time, eventually exhausting the PID namespace and causing `fork()` failures with "cannot allocate memory" errors despite free RAM. On `docker stop` or Kubernetes pod termination, the app ignores SIGTERM, so the runtime waits for `terminationGracePeriodSeconds` (default 30s) then sends SIGKILL — causing ungraceful shutdowns, dropped in-flight requests, and potential data corruption. The fix: use `tini` as PID 1 in the Dockerfile (`ENTRYPOINT ["/tini", "--"]`), or in Kubernetes set `shareProcessNamespace: true` with a proper init wrapper.

---

**Q3: A batch processing service on a shared host is causing latency spikes in a co-located API service during peak hours. Both run as systemd units. How do you isolate them without moving to separate hosts?**

> I'd use systemd's cgroup-based resource controls to isolate the batch service. First, check current usage with `systemd-cgtop`. Then edit the batch service unit file:
>
> ```ini
> [Service]
> CPUQuota=40%       # limit to 40% of one CPU
> CPUWeight=100      # lower weight vs API service at 1000
> IOWeight=100       # lower I/O priority
> MemoryMax=2G
> Nice=10            # lower scheduling priority
> ```
>
> For the API service, set `CPUWeight=1000` and `IOWeight=1000`. If the host has enough cores, add `CPUAffinity=` to pin each service to different physical cores, eliminating cache contention entirely. After `systemctl daemon-reload` and restarting both services, verify with `systemd-cgtop` and monitor API p99 latency. This is the same mechanism Kubernetes uses for resource limits — it's cgroups under the hood.

---

**Q4: During an incident, you need to reduce the priority of a runaway analytics process that's starving your web server, but you cannot restart either process. What's your exact approach?**

> I'd use `renice` and `ionice` to adjust priorities without restarting. First, identify PIDs: `pgrep -a analytics-worker` and `pgrep -a nginx`. Then:
>
> ```bash
> # Lower analytics priority to minimum
> renice +19 -p <analytics-PID>
>
> # Raise nginx priority (requires root)
> renice -5 -p <nginx-PID>
>
> # If disk I/O is the bottleneck:
> ionice -c 3 -p <analytics-PID>        # idle class — only gets I/O when nothing else needs it
> ionice -c 1 -n 0 -p <nginx-PID>       # real-time class
> ```
>
> Verify the effect with `top` and watch CPU distribution shift. This is a temporary mitigation — the permanent fix is adding `CPUQuota`, `IOWeight`, and `Nice` to the systemd unit files so limits survive restarts.

---

**Q5: You're investigating why a containerized application occasionally gets SIGKILL'd without any OOM event in the kernel logs. What are the possible causes and how do you distinguish between them?**

> There are several non-OOM reasons a container gets SIGKILL'd: (1) **Kubernetes liveness probe failure** — kubelet kills and restarts the pod after consecutive failures; check `kubectl describe pod` for "Liveness probe failed" events. (2) **CPU throttling causing probe timeout** — if the container is heavily CPU-throttled, the liveness probe HTTP call times out; check `container_cpu_cfs_throttled_seconds_total` in Prometheus. (3) **`terminationGracePeriodSeconds` exceeded** — during rolling updates, if the app doesn't handle SIGTERM and exit within the grace period, kubelet sends SIGKILL; check pod termination logs. (4) **Kubelet eviction** — the kubelet's eviction manager kills pods before the kernel OOM killer fires, based on `memory.available` thresholds; check `kubectl describe node` for eviction conditions and pressure taints. (5) **Container runtime issue** — check `journalctl -u containerd` for runtime errors. I'd distinguish these by correlating the kill timestamp with `kubectl get events --sort-by='.lastTimestamp'`, node-level metrics, and container runtime logs.

---


## 1.3 File System & Storage

### Concept Deep Dive

Linux uses a **Virtual File System (VFS)** layer — an abstraction that lets the kernel present a unified interface regardless of the underlying filesystem type (ext4, xfs, tmpfs, NFS, procfs). Everything is accessed through the same `open()`, `read()`, `write()` system calls.

---

### The Inode — The Real Identity of a File

A file in Linux has two parts:

- **Inode**: metadata (permissions, owner, timestamps, size, pointers to data blocks) — stored in the filesystem
- **Directory entry (dentry)**: the name-to-inode mapping — stored in the directory

```
Directory entry:          Inode #4821:              Data blocks:
"nginx.conf" → 4821  →   permissions: 644      →   [block 1024]
                          owner: root                [block 1025]
                          size: 2048 bytes           [block 1026]
                          mtime: 2026-03-01
                          links: 1
```

This separation is why:

- You can have multiple filenames (hard links) pointing to the same inode
- Renaming a file in the same filesystem is instant (just updates the dentry, inode unchanged)
- A file is only truly deleted when its link count reaches 0 **and** no process has it open

---

### Filesystem Comparison: ext4 vs xfs

| Feature | ext4 | xfs |
|---|---|---|
| Max file size | 16 TB | 8 EB |
| Max filesystem size | 1 EB | 8 EB |
| Best for | General purpose, small files | Large files, high throughput |
| Default on | Ubuntu, Debian | RHEL, Amazon Linux 2023 |
| Shrink support | Yes | **No — cannot shrink** |
| Inode allocation | Fixed at creation | Dynamic |

---

### The `/proc` and `/sys` Virtual Filesystems

`/proc` is not on disk — it is generated by the kernel in real-time. Every file read from `/proc` triggers a kernel function that returns current state. This is how `top`, `ps`, and `free` work.

```bash
# Key /proc paths
/proc/<PID>/cmdline      # command line arguments
/proc/<PID>/environ      # environment variables
/proc/<PID>/fd/          # open file descriptors
/proc/<PID>/maps         # memory mappings
/proc/<PID>/status       # human-readable process status
/proc/meminfo            # memory statistics
/proc/cpuinfo            # CPU information
/proc/loadavg            # load average
/proc/sys/               # tunable kernel parameters (sysctl)
```

---

### LVM — Logical Volume Manager

LVM adds a layer of abstraction between physical disks and filesystems, enabling online resizing, snapshots, and spanning volumes across multiple disks.

```
Physical Disks:    /dev/xvdb    /dev/xvdc    /dev/xvdd
                       │            │            │
Physical Volumes:  PV           PV           PV
                       └────────────┴────────────┘
                                    │
Volume Group:              VG "data-vg" (total pool)
                                    │
Logical Volumes:    LV "app-data"   LV "db-data"   LV "logs"
                         │               │              │
Filesystems:           ext4            xfs           ext4
Mount Points:       /opt/app        /var/lib/db    /var/log
```

```bash
# LVM inspection
pvs                                       # list physical volumes
vgs                                       # list volume groups
lvs                                       # list logical volumes

# Extend a logical volume online (no downtime)
lvextend -L +10G /dev/data-vg/app-data
xfs_growfs /opt/app                       # for xfs
resize2fs /dev/data-vg/app-data           # for ext4

# Create a snapshot (for backup without downtime)
lvcreate -L 5G -s -n app-data-snap /dev/data-vg/app-data
```

---

### Real-World Production Scenarios

**Scenario 1: Application logs show "No space left on device" but `df -h` shows 40% free space**

```bash
# Classic inode exhaustion — too many small files
df -i
# Output: /dev/xvda1   2097152  2097152  0  100% /

# Find the directory with millions of files
find / -xdev -printf '%h
' | sort | uniq -c | sort -rn | head -10

# Common culprits in production:
# - PHP session files in /var/lib/php/sessions/
# - Application temp files not being cleaned up
# - Log files split per-request (misconfigured logging)
# - Celery/queue worker result files not being purged

# Quick count in a suspected directory
ls /var/lib/php/sessions/ | wc -l

# Clean up (verify before deleting)
find /var/lib/php/sessions/ -type f -mtime +7 -delete

# Long-term fix: switch to xfs (dynamic inode allocation)
# Or at mkfs time: mkfs.ext4 -N 4000000 /dev/xvdb
```

**Scenario 2: A log file was deleted to free space, but `df -h` still shows the space as used**

```bash
# A process still has the file open — the inode is still referenced
# Space is freed only when the file descriptor is closed

# Find the process holding the deleted file
lsof | grep deleted
# Output: java  1234  appuser  10w  REG  202,1  5368709120  deleted /var/log/app/app.log

# Option 1: Restart the process (causes downtime)
systemctl restart myapp

# Option 2: Truncate via /proc — zero downtime!
# FD number from lsof output (column 4: "10w" = FD 10)
> /proc/1234/fd/10

# Option 3: Configure logrotate with copytruncate for future prevention
# copytruncate: copies the log then truncates the original (no restart needed)
```

**Scenario 3: EBS volume on an EC2 instance needs to be extended with zero downtime**

```bash
# Step 1: Extend the EBS volume via AWS CLI
aws ec2 modify-volume --volume-id vol-xxxx --size 100

# Step 2: Wait for modification to complete
aws ec2 describe-volumes-modifications --volume-id vol-xxxx

# Step 3: Extend the partition (if using a partition, not raw device)
lsblk
growpart /dev/nvme0n1 1       # extend partition 1

# Step 4: Extend the filesystem online (no unmount needed)
xfs_growfs /                  # for xfs
resize2fs /dev/nvme0n1p1      # for ext4

# Verify
df -h
```

---

### Interview Questions & Answers

**Q1: A production database server's `/var/lib/mysql` filesystem is 95% full. You cannot add a new disk immediately. Walk me through how you'd safely reclaim space without taking the database offline.**

> First, I'd identify what's consuming space: `du -sh /var/lib/mysql/* | sort -rh | head -20`. Common culprits are binary logs, InnoDB undo logs, or large temporary tables. For binary logs, I'd check retention in MySQL: `SHOW BINARY LOGS;` and purge safely: `PURGE BINARY LOGS BEFORE DATE_SUB(NOW(), INTERVAL 3 DAY);` — safe only after verifying replicas are caught up via `SHOW SLAVE STATUS`. I'd check for deleted-but-open files: `lsof | grep deleted | grep mysql` and truncate them via `/proc/<PID>/fd/<FD>` without restarting MySQL. I'd also check for core dumps in `/var/lib/mysql/` and remove them. If there are large `.ibd` files from dropped tables, I'd run `OPTIMIZE TABLE` on the largest ones during a low-traffic window. Throughout, I'd monitor `df -h` and MySQL error logs to ensure no new errors are introduced.

---

**Q2: You're migrating a service from ext4 to xfs for better performance with large files. The volume has 500GB of data. What are the risks and how do you execute this with minimal downtime?**

> The key risk is that **xfs cannot be shrunk** — if you over-provision, you're stuck. Also, xfs and ext4 have different tuning parameters, so small-file performance may regress. My approach: provision a new EBS volume with xfs (`mkfs.xfs /dev/nvme1n1`), then use `rsync -aHAXxv --numeric-ids --progress /data/ /mnt/new-volume/` to sync data while the service is live. I'd do a final sync with the service briefly stopped to catch the delta: stop service → final `rsync` → update `/etc/fstab` → mount new volume → start service. Total downtime is only the final rsync delta, typically seconds to minutes for 500GB. I'd keep the old ext4 volume detached (not deleted) for 48 hours as a rollback option. Post-migration, I'd benchmark with `fio` to confirm the expected performance improvement before decommissioning the old volume.

---

**Q3: A developer reports that after deleting large log files, the disk is still full and the application is still writing to the deleted file. Explain what's happening and provide two resolution paths with their trade-offs.**

> When a file is deleted in Linux, the directory entry (filename) is removed but the inode and data blocks are not freed until all file descriptors referencing that inode are closed. The application still has the file open, so it continues writing to it — the data is going to disk but the filename is gone.
>
> **Path 1 — Restart the process:** `systemctl restart myapp`. This closes all file descriptors, immediately freeing the disk space. Trade-off: causes downtime and drops in-flight requests.
>
> **Path 2 — Truncate via `/proc` (zero downtime):** Find the FD with `lsof | grep deleted | grep myapp`, note the PID and FD number, then run `> /proc/<PID>/fd/<FD>`. This truncates the file to zero bytes while the process keeps its file descriptor open and continues writing. Disk space is reclaimed immediately. Trade-off: the process continues writing to the same inode, so log rotation must be configured correctly going forward. This is the preferred approach for production systems where downtime is unacceptable.

---

**Q4: You need to take a consistent backup of a live PostgreSQL data directory on an LVM volume without stopping the database. How do you do this and what are the risks?**

> I'd use an LVM snapshot. First, ensure there's free space in the volume group (`vgs`). Then:
>
> ```bash
> # Create a point-in-time snapshot (instant operation)
> lvcreate -L 20G -s -n pgdata-snap /dev/data-vg/pgdata
>
> # Mount the snapshot read-only
> mount -o ro /dev/data-vg/pgdata-snap /mnt/backup
>
> # Stream backup to S3
> tar -czf - /mnt/backup | aws s3 cp - s3://my-bucket/pgdata-backup-$(date +%F).tar.gz
>
> # Unmount and remove snapshot
> umount /mnt/backup
> lvremove /dev/data-vg/pgdata-snap
> ```
>
> The risks: (1) **Snapshot space exhaustion** — if the live volume writes faster than the backup completes, the snapshot fills up and becomes invalid; monitor with `lvs` during backup. (2) **Filesystem consistency** — the snapshot is crash-consistent but not application-consistent; for PostgreSQL, use `pg_start_backup()` before the snapshot and `pg_stop_backup()` after. (3) **Performance impact** — LVM snapshots use CoW, so every write to the original volume during the snapshot period incurs extra I/O overhead.

---

**Q5: A developer accidentally ran `chmod -R 777 /etc` on a production server. The server is still running. What are the immediate risks and how do you remediate without a full rebuild?**

> The immediate risks are severe: (1) Any process or user can now modify critical config files (`/etc/passwd`, `/etc/sudoers`, `/etc/ssh/sshd_config`), enabling privilege escalation. (2) SSH host keys in `/etc/ssh/` are world-readable. (3) `sudo` and `su` may refuse to work — they check that `/etc/sudoers` is not world-writable.
>
> Remediation:
>
> ```bash
> # Step 1: Immediately restrict network access (Security Group / NACL)
>
> # Step 2: Restore correct permissions for critical files
> chmod 644 /etc/passwd /etc/group /etc/hosts /etc/hostname
> chmod 640 /etc/shadow /etc/gshadow
> chmod 440 /etc/sudoers
> chmod 600 /etc/ssh/ssh_host_*_key
> chmod 644 /etc/ssh/ssh_host_*_key.pub
> chmod 755 /etc/ssh /etc/init.d /etc/cron.d
>
> # Step 3: Use rpm/dpkg to restore all package-owned file permissions
> # RHEL / Amazon Linux:
> rpm -qa | xargs rpm --setperms
>
> # Ubuntu / Debian:
> dpkg --get-selections | awk '{print $1}' | xargs dpkg --verify 2>/dev/null
> ```
>
> After remediation, rotate all SSH host keys, audit `/var/log/auth.log` for access during the exposure window, and treat the instance as potentially compromised — plan a rebuild at the next maintenance window.


## 1.4 Users & Groups

### Concept Deep Dive

Linux access control is built on three identity primitives: **users**, **groups**, and **file permissions**. Every process runs as a user (UID) and one or more groups (GID). Every file has an owner UID and group GID. The kernel checks these at every file access.

---

### The Key Files

```bash
/etc/passwd    # user accounts (username, UID, GID, home, shell)
/etc/shadow    # hashed passwords (readable only by root)
/etc/group     # group definitions (groupname, GID, members)
/etc/gshadow   # group passwords (rarely used)
/etc/sudoers   # sudo privilege definitions (edit with visudo only!)
```

**`/etc/passwd` format:**

```
username:x:UID:GID:comment:home_dir:shell
appuser:x:1001:1001:Application User:/home/appuser:/bin/bash
nobody:x:65534:65534::/nonexistent:/usr/sbin/nologin
```

- `x` in the password field means the password is stored in `/etc/shadow`
- `/usr/sbin/nologin` or `/bin/false` as shell = service account, cannot log in interactively

**`/etc/shadow` format:**

```
username:$6$salt$hash:lastchange:mindays:maxdays:warndays:inactive:expire:
```

- `$6$` = SHA-512 hash; `$1$` = MD5 (insecure); `$2y$` = bcrypt
- `!` or `*` prefix = account locked

---

### Privilege Escalation — sudo Internals

When you run `sudo command`, the sudo binary (which has the SUID bit set, owned by root) checks `/etc/sudoers` to verify you have permission, then calls `setuid(0)` to become root before executing the command. This is why `/etc/sudoers` must never be world-writable.

```bash
# Always edit sudoers with visudo (validates syntax before saving)
visudo

# Common sudoers patterns
appuser ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart myapp
%devops ALL=(ALL) ALL                          # group devops can run anything
deploy  ALL=(ALL) NOPASSWD: /usr/bin/docker

# Check what sudo privileges a user has
sudo -l -U appuser
```

---

### PAM — Pluggable Authentication Modules

PAM is the authentication framework that sits between applications (SSH, sudo, login) and the actual authentication mechanisms (passwords, LDAP, MFA). Configuration is in `/etc/pam.d/`.

```bash
# PAM module types:
# auth     - verify identity (password, key, MFA)
# account  - check account validity (expired, locked)
# session  - setup/teardown session (mount home, set limits)
# password - update authentication tokens

# PAM control flags:
# required   - must succeed, but continue checking others
# requisite  - must succeed, stop immediately if fails
# sufficient - if succeeds, stop checking (if no prior required failed)
# optional   - result doesn't matter
```

---

### User Management Commands

```bash
# Create a service account (no login shell, no home directory)
useradd -r -s /usr/sbin/nologin -M appuser

# Create a regular user
useradd -m -s /bin/bash -G sudo,docker developer

# Lock / unlock an account
passwd -l username       # lock
passwd -u username       # unlock

# Check account expiry details
chage -l username

# Set password to never expire (for service accounts)
chage -M -1 username

# Add user to a group without removing existing groups
usermod -aG docker appuser

# Delete a user and their home directory
userdel -r username
```

---

### Real-World Production Scenarios

**Scenario 1: A service account's password expired, causing a cron job to fail silently at midnight**

```bash
# Check account status
chage -l serviceaccount
# Output: Password expires: Feb 28, 2026

# Fix: set password to never expire for service accounts
chage -M -1 serviceaccount

# Lock the password and use SSH keys or tokens instead
passwd -l serviceaccount

# Audit all accounts with expiring passwords
awk -F: '$5 != "" && $5 != "-1" {print $1, $5}' /etc/shadow
```

**Scenario 2: A developer needs temporary elevated access to debug a production issue, but you cannot give them permanent sudo**

```bash
# Option 1: Scoped sudo for specific commands only (least privilege)
echo "developer ALL=(ALL) NOPASSWD: /usr/bin/journalctl, /usr/bin/systemctl status *" \
  > /etc/sudoers.d/developer-debug
chmod 440 /etc/sudoers.d/developer-debug

# Remove after the incident
rm /etc/sudoers.d/developer-debug

# Option 2: AWS SSM Session Manager with IAM-controlled access
# No SSH needed, full audit trail in CloudTrail, time-limited via IAM policy

# Audit what they did
grep developer /var/log/auth.log
grep developer /var/log/secure    # RHEL
```

---

### Interview Questions & Answers

**Q1: A security audit finds that a containerized application is running as root inside the container. The team argues it's safe because it's "just a container." How do you respond and what's your remediation plan?**

> Running as root in a container is dangerous for several reasons. If the container runtime has a vulnerability (like the runc CVE-2019-5736), a root container can escape to the host. If the application is compromised, the attacker has root inside the container and can potentially access mounted volumes, the Docker socket, or exploit kernel vulnerabilities. The "it's just a container" argument ignores that containers share the host kernel — there is no hypervisor boundary.
>
> Remediation:
>
> ```dockerfile
> # Add a non-root user in the Dockerfile
> RUN useradd -r -u 1001 -g appgroup appuser
> USER appuser
> ```
>
> In the Kubernetes `securityContext`:
>
> ```yaml
> securityContext:
>   runAsNonRoot: true
>   runAsUser: 1001
>   allowPrivilegeEscalation: false
>   readOnlyRootFilesystem: true
> ```
>
> Enforce this cluster-wide with Pod Security Admission (`restricted` profile) or an OPA/Gatekeeper policy that rejects pods running as root.

---

**Q2: You're setting up a deployment automation system where a CI/CD pipeline needs to SSH into production servers to deploy. How do you implement this securely using Linux user and key management?**

> I'd create a dedicated `deploy` service account with a locked password: `useradd -r -s /bin/bash deploy && passwd -l deploy`. Generate a dedicated SSH key pair for the CI/CD system: `ssh-keygen -t ed25519 -C "cicd-deploy-key"`. Add the public key to `/home/deploy/.ssh/authorized_keys` on all target servers with a command restriction:
>
> ```
> command="/usr/local/bin/deploy-wrapper.sh",no-port-forwarding,no-X11-forwarding,no-agent-forwarding ssh-ed25519 AAAA...
> ```
>
> The `deploy-wrapper.sh` validates the deployment command before executing. Store the private key in AWS Secrets Manager — never in the repository. In `/etc/sudoers.d/deploy`, grant only the specific commands needed: `deploy ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart myapp`. Rotate the key quarterly and immediately on team member departure. Log all deploy actions via `auditd` rules on the deploy account.

---

**Q3: During an incident investigation, you need to find all files on a server that have the SUID bit set. Why is this important and what would you do with the results?**

> SUID files run with the file owner's privileges (typically root) regardless of who executes them. A malicious or vulnerable SUID binary is a privilege escalation vector.
>
> ```bash
> find / -perm -4000 -type f -ls 2>/dev/null
> ```
>
> I'd compare the results against a known-good baseline from the AMI or a clean instance. Legitimate SUID binaries include `passwd`, `sudo`, `ping`, `su`, `mount`. Any unexpected SUID binary — especially in `/tmp`, `/var`, user home directories, or application directories — is a red flag. For each unexpected finding, I'd check the file's hash against the package manager (`rpm -V <package>` or `dpkg --verify`), check its creation/modification time with `stat`, and check who owns it. In production, I'd use AWS Config or a file integrity monitoring tool (AIDE, Falco) to alert on new SUID files being created.

---

**Q4: A production incident reveals that a compromised application process was able to read `/etc/shadow`. How is this possible and how do you prevent it?**

> `/etc/shadow` should be readable only by root (`chmod 640`, owned by `root:shadow`). If an application process could read it, one of the following happened: (1) The process is running as root — remediate by running the app as a non-root user. (2) The file permissions were changed (e.g., by a bad `chmod -R` command) — check with `stat /etc/shadow` and restore with `chmod 640 /etc/shadow && chown root:shadow /etc/shadow`. (3) The process has `CAP_DAC_READ_SEARCH` capability set — check with `getcap /path/to/binary` and remove unnecessary capabilities with `setcap -r /path/to/binary`. (4) A container is running with `hostPID: true` or a privileged security context — audit all pod specs for `privileged: true` and `hostPID: true`. Prevention: enforce least-privilege process accounts, use Pod Security Admission in Kubernetes, and run `auditd` rules that alert on any read of `/etc/shadow` by non-root processes.



## 1.5 Package Management

### Concept Deep Dive

Package managers handle the installation, upgrade, and removal of software, including dependency resolution. Understanding them is critical for maintaining consistent, reproducible server configurations.

---

### The Two Major Ecosystems

| Ecosystem | Distros | Package Format | Low-Level Tool | High-Level Tool |
|---|---|---|---|---|
| Debian | Ubuntu, Debian | `.deb` | `dpkg` | `apt` / `apt-get` |
| Red Hat | RHEL, Amazon Linux, CentOS | `.rpm` | `rpm` | `yum` / `dnf` |

---

### How Package Installation Works Internally

```
apt install nginx
    │
    ▼
Resolve dependencies (nginx requires libpcre, openssl, etc.)
    │
    ▼
Download .deb packages from repository
    │
    ▼
Verify GPG signature (ensures package authenticity)
    │
    ▼
dpkg extracts files to correct locations
    │
    ▼
Run post-install scripts (create user, enable service, etc.)
    │
    ▼
Update package database (/var/lib/dpkg/status)
```

---

### Repository Configuration

```bash
# Ubuntu/Debian
/etc/apt/sources.list
/etc/apt/sources.list.d/*.list

# RHEL/Amazon Linux
/etc/yum.repos.d/*.repo

# Add a repository (Ubuntu)
echo "deb [signed-by=/usr/share/keyrings/nginx.gpg] http://nginx.org/packages/ubuntu focal nginx" \
  > /etc/apt/sources.list.d/nginx.list

# Import GPG key
curl -fsSL https://nginx.org/keys/nginx_signing.key | \
  gpg --dearmor -o /usr/share/keyrings/nginx.gpg

# Update package index
apt update

# List available versions of a package
apt-cache policy nginx
yum --showduplicates list nginx
```

---

### Holding Packages — Preventing Unintended Upgrades

```bash
# Ubuntu/Debian — hold a package at current version
apt-mark hold nginx
apt-mark showhold
apt-mark unhold nginx

# RHEL/Amazon Linux
yum versionlock add nginx
yum versionlock list
yum versionlock delete nginx
# Requires: yum install yum-plugin-versionlock
```

---

### Useful Diagnostic Commands

```bash
# Check which package owns a file
rpm -qf /usr/sbin/nginx                            # RHEL
dpkg -S /usr/sbin/nginx                            # Ubuntu

# List all files installed by a package
rpm -ql nginx
dpkg -L nginx

# Verify package file integrity (detect tampering)
rpm -V nginx                                       # RHEL — shows modified files
dpkg --verify nginx                                # Ubuntu

# Check package changelog
rpm -q --changelog nginx | head -30
apt-get changelog nginx

# List recently installed packages
rpm -qa --last | head -20                          # RHEL
grep " install " /var/log/dpkg.log | tail -20      # Ubuntu
```

---

### Real-World Production Scenarios

**Scenario 1: An `apt upgrade` on a production server upgraded the kernel and broke a custom kernel module**

```bash
# Prevention: hold the kernel package before any upgrade
apt-mark hold linux-image-$(uname -r)
apt-mark hold linux-headers-$(uname -r)

# Or hold all kernel packages generically
apt-mark hold linux-image-generic linux-headers-generic

# If already upgraded and broken:
# Boot into previous kernel from GRUB, then pin the working version

# Check installed kernels
dpkg --list | grep linux-image

# Remove the broken new kernel
apt remove linux-image-5.15.0-100-generic

# Best practice: test kernel upgrades on a non-production instance first
# Use AWS Systems Manager Patch Manager with a patch baseline
# that excludes kernel packages for production
```

---

**Scenario 2: A security vulnerability is found in OpenSSL. You need to patch 200 EC2 instances with zero downtime**

```bash
# Step 1: Identify affected instances
aws ssm run-command \
  --document-name "AWS-RunShellScript" \
  --parameters 'commands=["openssl version"]' \
  --targets "Key=tag:Environment,Values=production"

# Step 2: Use SSM Patch Manager with a maintenance window
# Create a patch baseline that includes the OpenSSL CVE
# Schedule the maintenance window during low-traffic hours

# Step 3: For zero downtime — rolling patch with ASG
# Suspend AZRebalance, then patch one AZ at a time
# Or use instance refresh with a custom pre-patch lifecycle hook

# Step 4: Verify after patching
aws ssm run-command \
  --document-name "AWS-RunShellScript" \
  --parameters 'commands=["openssl version && rpm -q openssl"]' \
  --targets "Key=tag:PatchGroup,Values=production"
```

---

**Scenario 3: A package installation fails due to a dependency conflict after a partial upgrade**

```bash
# Ubuntu/Debian
apt-get check           # check for broken dependencies
dpkg --audit            # audit unconfigured packages

# Fix broken dependencies
apt-get install -f      # -f = fix broken
dpkg --configure -a     # configure any unconfigured packages

# If a specific package is in a bad state
dpkg --remove --force-remove-reinstreq <broken-package>
apt-get install <broken-package>

# RHEL/Amazon Linux
yum clean all
yum makecache
yum install <package>

# Rebuild RPM database if corrupted
rpm --rebuilddb
```

---

### Interview Questions & Answers

**Q1: You need to install a specific version of a package on 300 servers and ensure it never gets accidentally upgraded during routine patching. How do you implement this at scale?**

> I'd bake the specific package version into the AMI using Packer — this ensures all new instances start with the correct version. For existing instances, I'd use AWS Systems Manager Run Command to install the pinned version: `yum install nginx-1.24.0-1.amzn2` (RHEL) or `apt install nginx=1.24.0-1` (Ubuntu). To prevent accidental upgrades, I'd configure version locking: `yum versionlock add nginx` (RHEL) or `apt-mark hold nginx` (Ubuntu) via another SSM Run Command. In the AWS Patch Manager patch baseline, I'd add nginx to the rejected patches list. I'd also add a compliance check — an SSM State Manager association that runs daily and reports any instances where the nginx version drifts from the expected value, alerting via SNS if drift is detected.

---

**Q2: A `yum update` fails halfway through on a production server, leaving the system in an inconsistent state. How do you recover?**

> `yum`/`dnf` uses transactions, so a failed update should be recoverable. First, check the yum transaction history: `yum history list` and `yum history info <transaction-id>`. If the transaction is incomplete, try to undo it: `yum history undo <transaction-id>`. If that fails, clean up stale locks: `rm -f /var/run/yum.pid` and `rm -f /var/lib/rpm/.rpm.lock`. Then run `rpm --rebuilddb` to rebuild the RPM database if it is corrupted. Check `rpm -Va` to verify all installed packages and identify any files that do not match their expected checksums. If the system is unstable, restore from the pre-patch AMI snapshot — which is why you always snapshot before patching production. Going forward, use SSM Patch Manager which handles rollback automatically and never patches without a snapshot.

---

**Q3: During a security review, you discover that a production server has packages installed that are not in your approved baseline. How do you identify them and enforce compliance going forward?**

> First, I'd generate the list of installed packages and compare against the approved baseline:
>
> ```bash
> # RHEL — export installed packages
> rpm -qa --queryformat '%{NAME}-%{VERSION}-%{RELEASE}.%{ARCH}
' | sort > /tmp/installed.txt
>
> # Ubuntu — export installed packages
> dpkg-query -W -f='${Package}=${Version}
' | sort > /tmp/installed.txt
>
> # Diff against approved baseline
> diff /tmp/approved-baseline.txt /tmp/installed.txt
> ```
>
> For packages not in the baseline, I'd investigate when they were installed and by whom: `rpm -qa --last` (RHEL) or `grep install /var/log/dpkg.log` (Ubuntu). For enforcement going forward, I'd use AWS Config with a custom rule that compares the installed package list against the approved baseline on a schedule and flags non-compliant instances. I'd also use SSM State Manager to enforce the baseline — any drift triggers an automatic remediation run. For new instances, the Packer AMI pipeline enforces the baseline at build time, so only approved packages are ever present from launch.

---

**Q4: A developer installs a package manually on a production server using `rpm -ivh --nodeps`. Two weeks later, the application starts crashing. How do you investigate whether the manual package install is the cause?**

> The `--nodeps` flag bypasses dependency checking, which means the package may have installed but its required libraries are missing or at the wrong version — causing crashes when the application tries to load them. I'd investigate:
>
> ```bash
> # Check when the package was installed
> rpm -qa --last | grep <package-name>
>
> # Verify the package's dependencies are satisfied
> rpm -qR <package-name>                  # list required dependencies
> rpm -V <package-name>                   # verify installed files
>
> # Check for missing shared libraries
> ldd /path/to/binary | grep "not found"
>
> # Check application crash logs for library errors
> journalctl -u myapp --since "2 weeks ago" | grep -i "error\|cannot\|failed"
>
> # Check if the package conflicts with existing packages
> rpm -q --conflicts <package-name>
> ```
>
> If the manual package is the cause, I'd remove it (`rpm -e <package-name>`), install it properly with dependency resolution (`yum install <package-name>`), and verify the application recovers. Going forward, I'd enforce that no one can use `rpm -ivh --nodeps` in production by adding an `auditd` rule that alerts on direct `rpm` invocations outside of the approved patch management process.

---

**Q5: Your team uses a private internal package repository. During a deployment, a server pulls a package from the public internet instead of your internal repo. What are the security implications and how do you prevent this?**

> This is a **dependency confusion** or **repo priority misconfiguration** attack surface. If a public package with the same name as your internal package exists, the package manager may pull the wrong one — potentially a malicious version. The security implications: (1) Untrusted code runs on production servers. (2) The package may not have been security-scanned. (3) It could be a supply chain attack vector.
>
> Prevention:
>
> ```bash
> # RHEL — set repo priority so internal repo always wins
> # /etc/yum.repos.d/internal.repo
> [internal]
> name=Internal Repository
> baseurl=https://repo.internal.company.com/
> enabled=1
> priority=1          # lower number = higher priority
> gpgcheck=1
> gpgkey=https://repo.internal.company.com/RPM-GPG-KEY
>
> # Disable all external repos on production servers
> yum-config-manager --disable \*
> yum-config-manager --enable internal
>
> # Ubuntu — pin internal repo with higher priority
> # /etc/apt/preferences.d/internal-pin
> Package: *
> Pin: origin repo.internal.company.com
> Pin-Priority: [PIN]
> ```
>
> At the network level, I'd use Security Groups and NACLs to block outbound access to public package repositories from production servers entirely, forcing all traffic through the internal repo or an approved proxy. I'd also enforce GPG signature verification (`gpgcheck=1`) so any unsigned or incorrectly signed package is rejected regardless of source.


## 1.6 Shell Scripting

### Concept Deep Dive

Shell scripting is the glue of DevOps automation. Bash is the default shell on most Linux systems and the language of choice for system automation, CI/CD scripts, and operational tooling.

**Script execution internals:**

When you run `./script.sh`, the kernel reads the **shebang** (`#!/bin/bash`) and executes the specified interpreter with the script as input. Without a shebang, the current shell interprets it — which can cause subtle bugs if the user's shell is `zsh` or `dash`.

**Critical script settings — always use these:**

```bash
#!/bin/bash
set -euo pipefail
# -e          : exit immediately on any error
# -u          : treat unset variables as errors (prevents silent bugs)
# -o pipefail : pipeline fails if any command in it fails
#               (without this: "false | true" returns 0)

# Trap for cleanup on exit
trap 'cleanup' EXIT
trap 'echo "Error on line $LINENO"; cleanup; exit 1' ERR

cleanup() {
    rm -f /tmp/lockfile
    # any other cleanup
}
```

**Variables and quoting — the most common source of bugs:**

```bash
# Always quote variables to handle spaces and special characters
FILE="my file.txt"
rm $FILE    # WRONG: treated as two arguments — "my" and "file.txt"
rm "$FILE"  # CORRECT: treated as one argument

# Arrays
SERVERS=("web-01" "web-02" "web-03")
for server in "${SERVERS[@]}"; do
    ssh "$server" "systemctl status nginx"
done

# Command substitution
CURRENT_DATE=$(date +%Y-%m-%d)    # preferred
CURRENT_DATE=`date +%Y-%m-%d`     # older style, avoid

# Arithmetic
COUNT=$((COUNT + 1))
```

**Exit codes — the contract between scripts:**

```bash
# Every command returns an exit code: 0 = success, non-zero = failure
if ! aws s3 cp file.txt s3://bucket/; then
    echo "Upload failed" >&2
    exit 1
fi

# Common exit codes
# 0   = success
# 1   = general error
# 2   = misuse of shell command
# 126 = command found but not executable
# 127 = command not found
# 130 = terminated by Ctrl+C (128 + 2)
```

**Text processing tools — the production toolkit:**

```bash
# grep — search
grep -r "ERROR" /var/log/app/            # recursive
grep -v "DEBUG" app.log                  # invert match
grep -E "ERROR|WARN" app.log             # extended regex
grep -c "ERROR" app.log                  # count matches

# awk — column processing
awk '{print $1, $4}' access.log          # print columns 1 and 4
awk -F: '{print $1}' /etc/passwd         # custom delimiter
awk '$9 == "500" {print $0}' access.log  # filter by column value
awk '{sum += $NF} END {print sum}' file  # sum last column

# sed — stream editing
sed 's/old/new/g' file                   # replace all occurrences
sed -i 's/DEBUG/INFO/g' config.yaml     # in-place edit
sed -n '10,20p' file                     # print lines 10-20
sed '/^#/d' config.conf                  # delete comment lines

# sort, uniq — frequency analysis
sort access.log | uniq -c | sort -rn | head -10
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10  # top IPs

# xargs — build commands from input
find /tmp -name "*.tmp" -mtime +7 | xargs rm -f
cat servers.txt | xargs -P 10 -I{} ssh {} "uptime"   # 10 parallel connections
```

---

### Real-World Production Scenarios

**Scenario 1: A deployment script fails halfway through, leaving the application in a broken state across 50 servers**

```bash
#!/bin/bash
set -euo pipefail

SERVERS=($(cat /etc/deploy/server-list.txt))
FAILED_SERVERS=()

deploy_server() {
    local server="$1"
    echo "Deploying to $server..."

    # Create backup before deploying
    ssh "$server" "cp -r /opt/app /opt/app.backup.$(date +%s)" || return 1

    # Deploy
    rsync -az --delete "dist/" "$server:/opt/app/" || return 1
    ssh "$server" "systemctl restart myapp" || return 1

    # Health check
    sleep 5
    if ! ssh "$server" "curl -sf http://localhost:8080/health"; then
        echo "Health check failed on $server, rolling back..."
        ssh "$server" "rm -rf /opt/app && mv /opt/app.backup.* /opt/app && systemctl restart myapp"
        return 1
    fi

    return 0
}

for server in "${SERVERS[@]}"; do
    if ! deploy_server "$server"; then
        FAILED_SERVERS+=("$server")
    fi
done

if [ ${#FAILED_SERVERS[@]} -gt 0 ]; then
    echo "Deployment failed on: ${FAILED_SERVERS[*]}" >&2
    exit 1
fi

echo "Deployment successful on all servers"
```

**Scenario 2: You need to parse nginx access logs in real-time to detect IPs making more than 100 requests per minute and block them automatically**

```bash
#!/bin/bash
set -euo pipefail

THRESHOLD=100
BLOCK_DURATION=3600
LOG_FILE="/var/log/nginx/access.log"
SNS_TOPIC="arn:aws:sns:us-east-1:123456789:alerts"

tail -f "$LOG_FILE" | awk '{print $1}' | \
while read -r ip; do
    COUNT=$(grep -c "^$ip " /tmp/ip_window 2>/dev/null || echo 0)

    if [ "$COUNT" -gt "$THRESHOLD" ]; then
        if ! iptables -C INPUT -s "$ip" -j DROP 2>/dev/null; then
            echo "Blocking $ip ($COUNT req/min)"
            iptables -I INPUT -s "$ip" -j DROP

            # Schedule unblock
            (sleep "$BLOCK_DURATION" && iptables -D INPUT -s "$ip" -j DROP) &

            # Alert via SNS
            aws sns publish \
              --topic-arn "$SNS_TOPIC" \
              --message "Blocked $ip at $(date) — $COUNT requests/min"
        fi
    fi
done
```

---

### Interview Questions & Answers

**Q1: You have a bash script that runs as part of a CI/CD pipeline. It deploys to production and has been working fine, but today it silently succeeded (exit code 0) even though the actual deployment failed. How does this happen and how do you fix it?**

> This is the classic `pipefail` problem. Without `set -o pipefail`, a pipeline like `deploy_app | tee deploy.log` returns the exit code of the last command (`tee`), not `deploy_app`. If `deploy_app` fails but `tee` succeeds, the pipeline returns 0. Similarly, without `set -e`, a failed command in the middle of a script is silently ignored and execution continues.
>
> The fix: add `set -euo pipefail` at the top of every script. Also audit for patterns like `command || true` which intentionally swallow errors — make sure each one is deliberate. For the CI/CD pipeline specifically, add an explicit health check after deployment as a separate step that independently verifies the deployment succeeded — never rely solely on the deploy script's exit code.

---

**Q2: Write a script that checks if a list of services are running on a remote server, restarts any that are down, and sends an alert if a service fails to restart after 3 attempts.**

```bash
#!/bin/bash
set -euo pipefail

SERVICES=("nginx" "myapp" "redis")
SERVER="$1"
MAX_RETRIES=3
SNS_TOPIC_ARN="arn:aws:sns:us-east-1:123456789:alerts"
LOCK_FILE="/tmp/service-check-${SERVER}.lock"

# Prevent concurrent runs
exec 9>"$LOCK_FILE"
flock -n 9 || { echo "Another instance is running"; exit 0; }
trap 'rm -f "$LOCK_FILE"' EXIT

check_and_restart() {
    local service="$1"

    if ssh -o ConnectTimeout=5 "$SERVER" "systemctl is-active --quiet $service"; then
        return 0
    fi

    echo "Service $service is down on $SERVER. Attempting restart..."

    for attempt in $(seq 1 $MAX_RETRIES); do
        ssh "$SERVER" "systemctl restart $service"
        sleep 10

        if ssh "$SERVER" "systemctl is-active --quiet $service"; then
            echo "Service $service restarted successfully on attempt $attempt"
            return 0
        fi

        echo "Restart attempt $attempt failed for $service"
    done

    # All retries failed — send alert
    MESSAGE="CRITICAL: Service $service on $SERVER failed to restart after $MAX_RETRIES attempts at $(date)"
    aws sns publish \
        --topic-arn "$SNS_TOPIC_ARN" \
        --subject "Service Failure: $service on $SERVER" \
        --message "$MESSAGE"

    return 1
}

FAILED=0
for service in "${SERVICES[@]}"; do
    check_and_restart "$service" || FAILED=$((FAILED + 1))
done

exit $FAILED
```

---

**Q3: A script that runs `find / -name "*.log" -delete` is accidentally executed on a production server. The application is still running and writing to those log files. What do you do to recover the logs?**

> Since the application still has the deleted log files open, the data is still accessible via `/proc`. I'd immediately:
>
> ```bash
> # Find the application's PID and open file descriptors
> lsof | grep deleted | grep ".log"
> # Output: myapp  1234  appuser  5w  REG  ...  deleted /var/log/app/app.log
>
> # Recover the file content from /proc
> cp /proc/1234/fd/5 /var/log/app/app.log.recovered
>
> # For multiple deleted log files
> lsof | grep deleted | grep ".log" | awk '{print $2, $4, $9}' | \
> while read pid fd filename; do
>     fd_num=$(echo $fd | tr -d 'rwu')
>     cp /proc/$pid/fd/$fd_num "${filename}.recovered"
>     echo "Recovered: ${filename}.recovered"
> done
> ```
>
> This recovers everything the process has written up to this moment. For logs already closed before I could act, I'd check if they were shipped to a centralized log system (CloudWatch Logs, OpenSearch) before deletion. Going forward, I'd implement real-time log shipping with Fluent Bit so local log deletion never means permanent data loss.

---

**Q4: You need to write a script that safely rotates credentials (database passwords) across 100 microservices with zero downtime. Each service reads its password from an environment variable set at startup. What's your approach?**

> Zero-downtime credential rotation with environment variables requires a dual-credential strategy:
>
> ```bash
> #!/bin/bash
> set -euo pipefail
>
> OLD_PASSWORD=$(aws secretsmanager get-secret-value \
>   --secret-id "$SECRET_ARN" --query SecretString --output text)
> NEW_PASSWORD=$(openssl rand -base64 32)
>
> # Step 1: Add new password to the database (keep old one active)
> mysql -h "$DB_HOST" -u admin -p"$ADMIN_PASS" \
>   -e "ALTER USER 'appuser'@'%' IDENTIFIED BY '$NEW_PASSWORD';"
>
> # Step 2: Update Secrets Manager with new value
> aws secretsmanager put-secret-value \
>   --secret-id "$SECRET_ARN" \
>   --secret-string "$NEW_PASSWORD"
>
> # Step 3: Rolling restart — services pick up new secret on start
> kubectl rollout restart deployment/myapp -n production
> kubectl rollout status deployment/myapp -n production --timeout=300s
>
> # Step 4: Verify new password works
> if mysql -h "$DB_HOST" -u appuser -p"$NEW_PASSWORD" -e "SELECT 1;" &>/dev/null; then
>     echo "New password verified — rotation complete"
> else
>     echo "FAILED: Rolling back"
>     aws secretsmanager put-secret-value \
>       --secret-id "$SECRET_ARN" \
>       --secret-string "$OLD_PASSWORD"
>     kubectl rollout undo deployment/myapp -n production
>     exit 1
> fi
> ```
>
> Key design decisions: (1) Update the database first so the new password is valid before any service tries to use it. (2) Use Kubernetes rolling restart so old pods still use the old password (which still works) while new pods use the new one. (3) Only remove the old password from the database after all pods are confirmed running with the new one. (4) Always have a rollback path.

---

**Q5: A cron job script has been running for 6 months without issues. Today it started failing with "Argument list too long." What caused this and how do you fix it without rewriting the entire script?**

> The error `Argument list too long` (E2BIG) occurs when the total length of command arguments exceeds the kernel's `ARG_MAX` limit (typically 2MB). After 6 months, the number of files being passed to a command like `rm *.log` has grown beyond this limit — the shell expands the glob before passing it to the command, and the expanded list is now too large.
>
> **Diagnosis:**
>
> ```bash
> getconf ARG_MAX
> ls /var/log/app/*.log | wc -l
> ```
>
> **Fix — replace glob expansion with `find | xargs`:**
>
> ```bash
> # BEFORE (broken):
> rm /var/log/app/*.log
>
> # AFTER (fixed — xargs batches arguments automatically):
> find /var/log/app/ -name "*.log" -print0 | xargs -0 rm -f
>
> # For processing files in parallel:
> find /data/input/ -name "*.csv" -print0 | xargs -0 -P 4 -I{} ./process.sh {}
> # -P 4       = 4 parallel workers
> # -print0/-0 = null-delimited, handles filenames with spaces safely
> ```
>
> The `-print0` and `-0` flags are critical in production — they handle filenames containing spaces or special characters that would otherwise break the pipeline.

---

