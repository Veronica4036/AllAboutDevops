# Day 3: Linux Fundamentals Part 2 🔐

*"File Systems, Permissions & Process Management - The Linux Security Model"*

**Estimated Time**: 3 hours  
**Difficulty**: Beginner to Intermediate  
**Prerequisites**: Day 2 completed, comfortable with basic Linux commands

---

## 🎯 Learning Objectives

By the end of today, you will:
- Master Linux file permissions and ownership
- Understand and manage Linux processes
- Monitor system performance and resources
- Install and manage software packages
- Configure environment variables
- Use advanced text processing tools (grep, sed, awk)
- **Interview Ready**: Demonstrate system administration skills confidently

---

## 📖 Theory Section: The Linux Security Model

### Understanding File Permissions: The Digital Bouncer System

Imagine Linux file permissions like a nightclub with three types of people:
- **Owner** (the club owner): Can do anything
- **Group** (VIP members): Have special privileges
- **Others** (general public): Limited access

Every file and directory has three types of permissions for each category:
- **Read (r)**: Can view the content
- **Write (w)**: Can modify the content
- **Execute (x)**: Can run the file (for programs) or enter the directory

### The Permission String Decoder

When you run `ls -l`, you see something like this:
```
-rwxr-xr-- 1 alice developers 1024 Jan 15 10:30 deploy.sh
```

Let's decode this:
```
-rwxr-xr-- 1 alice developers 1024 Jan 15 10:30 deploy.sh
│││││││││  │ │     │          │    │           │
│││││││││  │ │     │          │    │           └── File name
│││││││││  │ │     │          │    └── Last modified
│││││││││  │ │     │          └── File size
│││││││││  │ │     └── Group owner
│││││││││  │ └── User owner
│││││││││  └── Number of hard links
│││││││││
│││││││└└── Others permissions (r--)
│││││└└── Group permissions (r-x)
│││└└└── Owner permissions (rwx)
└── File type (- = regular file, d = directory, l = link)
```

### Real-World Permission Examples

**Web Server Files**:
```bash
# HTML files: readable by everyone, writable by owner
-rw-r--r-- 1 www-data www-data 2048 index.html

# CGI scripts: executable by web server, not readable by others
-rwxr-x--- 1 www-data www-data 1024 process.cgi

# Configuration files: readable/writable by owner only
-rw------- 1 root root 512 database.conf
```

**Why This Matters in DevOps**:
- **Security**: Prevent unauthorized access to sensitive files
- **Functionality**: Ensure scripts can execute and services can read configs
- **Compliance**: Meet security standards and audit requirements

### 💡 Did You Know?

**The Great AWS S3 Bucket Leak of 2017**: Many companies accidentally exposed sensitive data because they didn't understand permission models. Proper file permissions are your first line of defense against data breaches. In the cloud era, understanding permissions can literally save your company millions!

---

## 🔧 Mastering File Permissions

### Viewing Permissions

```bash
# Detailed listing with permissions
ls -l

# Show permissions for specific file
ls -l filename.txt

# Show directory permissions
ls -ld /path/to/directory

# Show permissions in octal format
stat -c "%a %n" filename.txt
# Output: 644 filename.txt
```

### Understanding Octal Permissions

Permissions can be represented as numbers:
- **Read (r)**: 4
- **Write (w)**: 2  
- **Execute (x)**: 1

**Common Permission Combinations**:
```
7 = 4+2+1 = rwx (read, write, execute)
6 = 4+2   = rw- (read, write)
5 = 4+1   = r-x (read, execute)
4 = 4     = r-- (read only)
0 = 0     = --- (no permissions)
```

**Real-World Examples**:
```bash
# 755: Owner can do everything, others can read and execute
chmod 755 script.sh
# Equivalent to: rwxr-xr-x

# 644: Owner can read/write, others can only read
chmod 644 config.txt
# Equivalent to: rw-r--r--

# 600: Owner can read/write, no access for others
chmod 600 private_key.pem
# Equivalent to: rw-------

# 777: Everyone can do everything (DANGEROUS!)
chmod 777 file.txt
# Equivalent to: rwxrwxrwx
```

### Changing Permissions

**Using Symbolic Mode**:
```bash
# Add execute permission for owner
chmod u+x script.sh

# Remove write permission for group and others
chmod go-w sensitive_file.txt

# Add read permission for everyone
chmod a+r public_file.txt

# Set exact permissions
chmod u=rwx,g=rx,o=r script.sh
```

**Using Octal Mode**:
```bash
# Set permissions using numbers
chmod 755 script.sh
chmod 644 config.txt
chmod 600 private_file.txt
```

### Changing Ownership

```bash
# Change owner
sudo chown alice file.txt

# Change owner and group
sudo chown alice:developers file.txt

# Change ownership recursively
sudo chown -R alice:developers /path/to/directory

# Change only group
sudo chgrp developers file.txt
```

### Real-World DevOps Scenario: Securing a Web Application

```bash
# Set up proper permissions for a web application
sudo mkdir -p /var/www/myapp/{public,private,logs}

# Public files: readable by web server and public
sudo chmod 755 /var/www/myapp/public
sudo chmod 644 /var/www/myapp/public/*.html

# Private files: accessible only by application
sudo chmod 700 /var/www/myapp/private
sudo chmod 600 /var/www/myapp/private/*

# Log files: writable by application, readable by admin
sudo chmod 755 /var/www/myapp/logs
sudo chmod 644 /var/www/myapp/logs/*.log

# Set proper ownership
sudo chown -R www-data:www-data /var/www/myapp
```

---

## ⚙️ Process Management: Controlling the Digital Workforce

### Understanding Processes

Think of processes like employees in a company:
- Each has a unique ID (PID - Process ID)
- They have a boss (Parent Process ID - PPID)
- They consume resources (CPU, memory)
- They can be promoted, demoted, or fired

### Viewing Processes

**`ps` - Process Status**:
```bash
# Show processes for current user
ps

# Show all processes with detailed info
ps aux

# Show process tree (family relationships)
ps auxf

# Show processes for specific user
ps -u alice

# Show processes by name
ps aux | grep apache2
```

**Understanding `ps aux` Output**:
```
USER  PID %CPU %MEM    VSZ   RSS TTY   STAT START   TIME COMMAND
alice 1234  2.5  1.2  45678  9876 pts/0 S    10:30   0:05 python app.py
│     │     │    │     │     │    │     │    │       │    │
│     │     │    │     │     │    │     │    │       │    └── Command
│     │     │    │     │     │    │     │    │       └── CPU time used
│     │     │    │     │     │    │     │    └── Start time
│     │     │    │     │     │    │     └── Process state
│     │     │    │     │     │    └── Terminal
│     │     │    │     │     └── Physical memory used (KB)
│     │     │    │     └── Virtual memory used (KB)
│     │     │    └── Memory percentage
│     │     └── CPU percentage
│     └── Process ID
└── User owner
```

**Process States**:
- **R**: Running
- **S**: Sleeping (waiting for event)
- **D**: Uninterruptible sleep (usually I/O)
- **Z**: Zombie (finished but not cleaned up)
- **T**: Stopped

**`top` - Real-Time Process Monitor**:
```bash
# Interactive process monitor
top

# Key commands in top:
# q: Quit
# k: Kill process (enter PID)
# M: Sort by memory usage
# P: Sort by CPU usage
# 1: Show individual CPU cores
```

**`htop` - Enhanced Process Monitor** (if available):
```bash
# More user-friendly version of top
htop

# Features:
# - Color-coded display
# - Mouse support
# - Tree view
# - Easy process killing
```

### Managing Processes

**Starting Processes**:
```bash
# Run in foreground
python app.py

# Run in background
python app.py &

# Run and detach from terminal
nohup python app.py &

# Run with specific priority (nice value)
nice -n 10 python app.py
```

**Stopping Processes**:
```bash
# Graceful termination (SIGTERM)
kill 1234

# Force termination (SIGKILL)
kill -9 1234

# Kill by name
killall python

# Kill processes matching pattern
pkill -f "python app.py"
```

**Signal Types**:
```bash
# Common signals
kill -TERM 1234  # Graceful shutdown (default)
kill -KILL 1234  # Force kill (cannot be ignored)
kill -HUP 1234   # Hang up (often used to reload config)
kill -USR1 1234  # User-defined signal 1
kill -STOP 1234  # Pause process
kill -CONT 1234  # Resume paused process
```

### Background Job Management

```bash
# Start job in background
python long_running_script.py &

# List background jobs
jobs

# Bring job to foreground
fg %1

# Send job to background
bg %1

# Detach job from terminal
disown %1
```

### Real-World Example: Managing a Web Server

```bash
# Check if web server is running
ps aux | grep apache2

# Start web server
sudo systemctl start apache2

# Check status
sudo systemctl status apache2

# Stop web server
sudo systemctl stop apache2

# Restart web server
sudo systemctl restart apache2

# Reload configuration without stopping
sudo systemctl reload apache2

# Enable auto-start on boot
sudo systemctl enable apache2
```

---

## 📊 System Monitoring: Keeping Your Finger on the Pulse

### CPU and Memory Monitoring

**`free` - Memory Usage**:
```bash
# Show memory usage
free

# Human-readable format
free -h

# Output explanation:
#               total        used        free      shared  buff/cache   available
# Mem:           7.7G        2.1G        3.2G        180M        2.4G        5.2G
# Swap:          2.0G          0B        2.0G
```

**`df` - Disk Space Usage**:
```bash
# Show disk usage
df

# Human-readable format
df -h

# Show inode usage
df -i

# Example output:
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/sda1        20G  8.5G   11G  45% /
# /dev/sda2       100G   45G   50G  48% /home
```

**`du` - Directory Usage**:
```bash
# Show directory sizes
du -h

# Show only top-level directories
du -h --max-depth=1

# Show largest directories first
du -h | sort -hr

# Find largest files
find . -type f -exec du -h {} + | sort -hr | head -10
```

### Network Monitoring

**`netstat` - Network Connections**:
```bash
# Show all connections
netstat -a

# Show listening ports
netstat -l

# Show TCP connections with process info
netstat -tulpn

# Show only specific port
netstat -tulpn | grep :80
```

**`ss` - Modern Network Tool**:
```bash
# Show all sockets
ss -a

# Show listening TCP sockets
ss -tl

# Show process using specific port
ss -tulpn | grep :22
```

### System Load and Performance

**`uptime` - System Load**:
```bash
uptime
# Output: 10:30:15 up 5 days, 2:15, 3 users, load average: 0.15, 0.25, 0.30
#         │        │              │       │
#         │        │              │       └── Load averages (1, 5, 15 minutes)
#         │        │              └── Number of logged-in users
#         │        └── System uptime
#         └── Current time
```

**Load Average Interpretation**:
- **< 1.0**: System is not busy
- **= 1.0**: System is fully utilized but not overloaded
- **> 1.0**: System is overloaded (tasks waiting for CPU)

**`iostat` - I/O Statistics** (if available):
```bash
# Show I/O statistics
iostat

# Continuous monitoring every 2 seconds
iostat 2

# Show extended statistics
iostat -x
```

---

## 📦 Package Management: Installing and Managing Software

### Understanding Package Managers

Different Linux distributions use different package managers:
- **Ubuntu/Debian**: `apt` (Advanced Package Tool)
- **CentOS/RHEL/Fedora**: `yum` or `dnf`
- **SUSE**: `zypper`
- **Arch**: `pacman`

### APT (Ubuntu/Debian)

**Basic Package Operations**:
```bash
# Update package list
sudo apt update

# Upgrade all packages
sudo apt upgrade

# Install a package
sudo apt install nginx

# Install multiple packages
sudo apt install nginx mysql-server php

# Remove a package
sudo apt remove nginx

# Remove package and configuration files
sudo apt purge nginx

# Remove unused packages
sudo apt autoremove
```

**Searching and Information**:
```bash
# Search for packages
apt search nginx

# Show package information
apt show nginx

# List installed packages
apt list --installed

# Show package dependencies
apt depends nginx
```

### YUM/DNF (CentOS/RHEL/Fedora)

```bash
# Update package list
sudo yum update
# or
sudo dnf update

# Install package
sudo yum install nginx
# or
sudo dnf install nginx

# Remove package
sudo yum remove nginx
# or
sudo dnf remove nginx

# Search packages
yum search nginx
# or
dnf search nginx
```

### Real-World DevOps Package Installation

```bash
# Install essential DevOps tools
sudo apt update
sudo apt install -y \
    curl \
    wget \
    git \
    vim \
    htop \
    tree \
    unzip \
    software-properties-common \
    apt-transport-https \
    ca-certificates \
    gnupg \
    lsb-release

# Install Docker (example of adding external repository)
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io
```

---

## 🌍 Environment Variables: Configuring Your System

### Understanding Environment Variables

Environment variables are like global settings for your system and applications. They're key-value pairs that programs can read to modify their behavior.

### Viewing Environment Variables

```bash
# Show all environment variables
env

# Show specific variable
echo $HOME
echo $PATH
echo $USER

# Show variable with default value if not set
echo ${JAVA_HOME:-/usr/lib/jvm/default}
```

### Setting Environment Variables

**Temporary (current session only)**:
```bash
# Set variable
export MY_VAR="Hello World"

# Use variable
echo $MY_VAR

# Set multiple variables
export DB_HOST="localhost" DB_PORT="5432" DB_NAME="myapp"
```

**Permanent (for user)**:
```bash
# Edit user's bash profile
nano ~/.bashrc

# Add at the end:
export JAVA_HOME="/usr/lib/jvm/java-11-openjdk"
export PATH="$PATH:$JAVA_HOME/bin"
export DB_CONNECTION_STRING="postgresql://localhost:5432/myapp"

# Reload configuration
source ~/.bashrc
```

**System-wide (for all users)**:
```bash
# Edit system environment
sudo nano /etc/environment

# Add variables:
JAVA_HOME="/usr/lib/jvm/java-11-openjdk"
EDITOR="vim"
```

### Important Environment Variables

**`PATH`** - Where the system looks for commands:
```bash
# Show current PATH
echo $PATH

# Add directory to PATH
export PATH="$PATH:/opt/myapp/bin"

# Add to beginning of PATH (higher priority)
export PATH="/opt/myapp/bin:$PATH"
```

**`HOME`** - User's home directory:
```bash
echo $HOME
# Output: /home/username
```

**`USER`** - Current username:
```bash
echo $USER
# Output: username
```

**`PWD`** - Current working directory:
```bash
echo $PWD
# Output: /current/directory/path
```

### Real-World DevOps Environment Setup

```bash
# Create environment configuration for a web application
nano ~/.bashrc

# Add these lines:
# Application settings
export APP_ENV="development"
export APP_DEBUG="true"
export APP_PORT="3000"

# Database settings
export DB_HOST="localhost"
export DB_PORT="5432"
export DB_NAME="myapp_dev"
export DB_USER="developer"
export DB_PASSWORD="dev_password"

# AWS settings
export AWS_REGION="us-west-2"
export AWS_DEFAULT_REGION="us-west-2"

# Docker settings
export DOCKER_HOST="unix:///var/run/docker.sock"

# Development tools
export EDITOR="vim"
export BROWSER="firefox"

# Reload configuration
source ~/.bashrc
```

---

## 🔍 Advanced Text Processing: The DevOps Trinity

### grep - The Pattern Hunter

**Basic Usage**:
```bash
# Search for pattern in file
grep "error" logfile.txt

# Case-insensitive search
grep -i "error" logfile.txt

# Search recursively in directories
grep -r "TODO" src/

# Show line numbers
grep -n "error" logfile.txt

# Show lines that DON'T match
grep -v "debug" logfile.txt

# Count matching lines
grep -c "error" logfile.txt
```

**Advanced grep**:
```bash
# Multiple patterns
grep -E "error|warning|critical" logfile.txt

# Word boundaries (exact word match)
grep -w "app" logfile.txt

# Show context (lines before and after)
grep -A 3 -B 3 "error" logfile.txt

# Search in compressed files
zgrep "error" logfile.gz

# Search for files containing pattern
grep -l "error" *.log
```

### sed - The Stream Editor

**Basic Substitution**:
```bash
# Replace first occurrence per line
sed 's/old/new/' file.txt

# Replace all occurrences
sed 's/old/new/g' file.txt

# Replace and save to new file
sed 's/old/new/g' file.txt > newfile.txt

# Edit file in place
sed -i 's/old/new/g' file.txt

# Replace only on specific lines
sed '2,5s/old/new/g' file.txt
```

**Advanced sed**:
```bash
# Delete lines containing pattern
sed '/pattern/d' file.txt

# Insert line after pattern
sed '/pattern/a\New line to insert' file.txt

# Replace using variables
OLD="localhost"
NEW="production.server.com"
sed "s/$OLD/$NEW/g" config.txt
```

### awk - The Data Processor

**Basic Usage**:
```bash
# Print specific columns
awk '{print $1, $3}' file.txt

# Print lines longer than 80 characters
awk 'length > 80' file.txt

# Sum numbers in first column
awk '{sum += $1} END {print sum}' numbers.txt

# Print with custom separator
awk -F',' '{print $2}' csvfile.csv
```

**Advanced awk**:
```bash
# Process log files
awk '$9 == 404 {print $1, $7}' access.log

# Calculate averages
awk '{sum += $3; count++} END {print sum/count}' data.txt

# Format output
awk '{printf "%-20s %10.2f\n", $1, $2}' file.txt
```

### Real-World Log Analysis Example

```bash
# Analyze Apache access logs
LOG_FILE="/var/log/apache2/access.log"

# Top 10 IP addresses by request count
awk '{print $1}' $LOG_FILE | sort | uniq -c | sort -nr | head -10

# Top 10 requested pages
awk '{print $7}' $LOG_FILE | sort | uniq -c | sort -nr | head -10

# Count HTTP status codes
awk '{print $9}' $LOG_FILE | sort | uniq -c | sort -nr

# Find all 404 errors with timestamps
grep " 404 " $LOG_FILE | awk '{print $4, $5, $7}' | sed 's/\[//g'

# Calculate bandwidth usage (assuming $10 is bytes)
awk '{sum += $10} END {print "Total bytes:", sum, "MB:", sum/1024/1024}' $LOG_FILE
```

---

## 🎯 Interview Preparation: System Administration Scenarios

### Question 1: Permission Troubleshooting

**Interviewer**: "A user reports they can't execute a script they just downloaded. What steps would you take to troubleshoot this?"

**How to Think Through This**:
1. Check current permissions
2. Identify what permissions are needed
3. Verify ownership
4. Consider security implications

**Sample Answer**:
```bash
# First, check current permissions
ls -l script.sh

# If it shows something like: -rw-r--r-- (no execute permission)
# Add execute permission for the owner
chmod u+x script.sh

# If it's owned by another user, check if user is in the right group
groups username

# If needed, change ownership (with proper authorization)
sudo chown username:usergroup script.sh

# Verify the fix
ls -l script.sh
./script.sh
```

**Follow-up Explanation**: "I always start by checking current permissions with `ls -l`. The most common issue is missing execute permission. I'd add it with `chmod u+x` for the owner only, or `chmod 755` if it needs to be executable by everyone. I always consider security - scripts should have minimal necessary permissions."

### Question 2: Process Investigation

**Interviewer**: "The system is running slowly. How would you identify which process is causing the issue?"

**Sample Answer**:
```bash
# Check overall system load
uptime
top

# Identify CPU-intensive processes
ps aux --sort=-%cpu | head -10

# Check memory usage
ps aux --sort=-%mem | head -10
free -h

# Check I/O wait
iostat 1 5

# Investigate specific process
ps -ef | grep [process_name]
lsof -p [PID]  # See what files the process has open

# Check system resources
df -h  # Disk space
netstat -tulpn  # Network connections
```

**Explanation**: "I use a systematic approach: overall system health first with `uptime` and `top`, then drill down to specific processes. I check both CPU and memory usage, and don't forget about I/O wait which can cause perceived slowness."

### Question 3: Log Analysis Challenge

**Interviewer**: "You need to find all failed login attempts from the last 24 hours. How would you do this?"

**Sample Answer**:
```bash
# Check authentication logs
sudo grep "Failed password" /var/log/auth.log

# Filter for last 24 hours (assuming today's date)
sudo grep "$(date '+%b %d')" /var/log/auth.log | grep "Failed password"

# More sophisticated - last 24 hours regardless of date change
sudo grep "Failed password" /var/log/auth.log | \
  awk -v date="$(date -d '1 day ago' '+%b %d')" '$1 $2 >= date'

# Count failed attempts by IP
sudo grep "Failed password" /var/log/auth.log | \
  awk '{print $(NF-3)}' | sort | uniq -c | sort -nr

# Find repeated offenders (more than 5 attempts)
sudo grep "Failed password" /var/log/auth.log | \
  awk '{print $(NF-3)}' | sort | uniq -c | awk '$1 > 5'
```

**Explanation**: "I start with the authentication log which records login attempts. I use `grep` to filter for failed passwords, then `awk` to extract and count IP addresses. The pipeline approach lets me identify patterns and potential security threats."

### Question 4: Disk Space Emergency

**Interviewer**: "You get an alert that a production server is at 95% disk usage. Walk me through your response."

**Sample Answer**:
```bash
# Immediate assessment
df -h  # See which filesystem is full

# Find largest directories
du -sh /* | sort -hr | head -10

# If /var is full, check logs
du -sh /var/* | sort -hr
find /var/log -name "*.log" -size +100M -ls

# Check for large temporary files
find /tmp -size +100M -ls
find /var/tmp -size +100M -ls

# Look for old log files that can be compressed or removed
find /var/log -name "*.log" -mtime +30 -ls

# Check for core dumps
find / -name "core.*" -size +100M 2>/dev/null

# Clean up safely
# Compress old logs
sudo gzip /var/log/*.log.1

# Remove old compressed logs
sudo find /var/log -name "*.gz" -mtime +90 -delete

# Clear temporary files
sudo rm -rf /tmp/*
```

**Explanation**: "This is a critical situation requiring immediate action. I start with `df -h` to identify the full filesystem, then use `du` to find the largest consumers. I focus on safe cleanup - logs, temp files, and old data. I never delete files I don't understand, and I always verify there's adequate free space before and after cleanup."

### Question 5: Environment Configuration

**Interviewer**: "How would you set up environment variables for a multi-environment application (dev, staging, production)?"

**Sample Answer**:
```bash
# Create environment-specific configuration files
sudo mkdir -p /etc/myapp/{dev,staging,prod}

# Development environment
sudo tee /etc/myapp/dev/env.conf << EOF
export APP_ENV="development"
export DB_HOST="dev-db.internal"
export DB_NAME="myapp_dev"
export LOG_LEVEL="debug"
export API_URL="https://dev-api.company.com"
EOF

# Staging environment
sudo tee /etc/myapp/staging/env.conf << EOF
export APP_ENV="staging"
export DB_HOST="staging-db.internal"
export DB_NAME="myapp_staging"
export LOG_LEVEL="info"
export API_URL="https://staging-api.company.com"
EOF

# Production environment
sudo tee /etc/myapp/prod/env.conf << EOF
export APP_ENV="production"
export DB_HOST="prod-db.internal"
export DB_NAME="myapp_prod"
export LOG_LEVEL="error"
export API_URL="https://api.company.com"
EOF

# Create a script to load environment
sudo tee /usr/local/bin/load-env << 'EOF'
#!/bin/bash
ENV=${1:-dev}
if [ -f "/etc/myapp/$ENV/env.conf" ]; then
    source "/etc/myapp/$ENV/env.conf"
    echo "Loaded $ENV environment"
else
    echo "Environment $ENV not found"
    exit 1
fi
EOF

sudo chmod +x /usr/local/bin/load-env

# Usage:
# source load-env dev
# source load-env staging
# source load-env prod
```

**Explanation**: "I separate environment configurations into different files to avoid mixing sensitive production data with development settings. The script approach allows easy switching between environments and reduces the risk of using wrong configurations."

---

## 🎮 Hands-On Challenge: System Administration Simulation

**Time**: 45 minutes  
**Scenario**: You're the new DevOps engineer at "TechCorp" and need to set up a secure, monitored web server environment.

### Mission 1: Secure File Setup (15 minutes)

```bash
# Create application structure
sudo mkdir -p /opt/webapp/{bin,config,logs,data}

# Create application user
sudo useradd -r -s /bin/false webapp

# Set proper ownership and permissions
sudo chown -R webapp:webapp /opt/webapp
sudo chmod 755 /opt/webapp
sudo chmod 750 /opt/webapp/{bin,config}
sudo chmod 755 /opt/webapp/logs
sudo chmod 700 /opt/webapp/data

# Create configuration files with proper permissions
sudo touch /opt/webapp/config/app.conf
sudo chmod 640 /opt/webapp/config/app.conf

# Create log files
sudo touch /opt/webapp/logs/{access.log,error.log}
sudo chmod 644 /opt/webapp/logs/*.log
```

### Mission 2: Process Monitoring Setup (15 minutes)

```bash
# Create a monitoring script
sudo tee /opt/webapp/bin/monitor.sh << 'EOF'
#!/bin/bash

LOG_FILE="/opt/webapp/logs/monitor.log"
DATE=$(date '+%Y-%m-%d %H:%M:%S')

# Check system resources
CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
MEM_USAGE=$(free | grep Mem | awk '{printf "%.1f", $3/$2 * 100.0}')
DISK_USAGE=$(df / | tail -1 | awk '{print $5}' | cut -d'%' -f1)

# Log the metrics
echo "$DATE - CPU: ${CPU_USAGE}%, Memory: ${MEM_USAGE}%, Disk: ${DISK_USAGE}%" >> $LOG_FILE

# Alert if usage is high
if [ $(echo "$CPU_USAGE > 80" | bc -l) -eq 1 ]; then
    echo "$DATE - ALERT: High CPU usage: ${CPU_USAGE}%" >> $LOG_FILE
fi

if [ $(echo "$MEM_USAGE > 80" | bc -l) -eq 1 ]; then
    echo "$DATE - ALERT: High memory usage: ${MEM_USAGE}%" >> $LOG_FILE
fi

if [ $DISK_USAGE -gt 80 ]; then
    echo "$DATE - ALERT: High disk usage: ${DISK_USAGE}%" >> $LOG_FILE
fi
EOF

sudo chmod +x /opt/webapp/bin/monitor.sh
sudo chown webapp:webapp /opt/webapp/bin/monitor.sh
```

### Mission 3: Log Analysis Tools (15 minutes)

```bash
# Create log analysis script
sudo tee /opt/webapp/bin/analyze_logs.sh << 'EOF'
#!/bin/bash

LOG_DIR="/opt/webapp/logs"
ACCESS_LOG="$LOG_DIR/access.log"
ERROR_LOG="$LOG_DIR/error.log"

echo "=== Log Analysis Report ==="
echo "Generated: $(date)"
echo

# Simulate some log entries for testing
Y-%m-%d %H:%M:%S') 192.168.1.100 GET /index.html 200 1024" >> $ACCESS_LOG
echo "$(date '+%Y-%m-%d %H:%M:%S') 192.168.1.101 GET /api/users 404 512" >> $ACCESS_LOG
echo "$(date '+%Y-%m-%d %H:%M:%S') 192.168.1.102 POST /api/login 500 256" >> $ACCESS_LOG
echo "$(date '+%Y-%m-%d %H:%M:%S') ERROR: Database connection failed" >> $ERROR_LOG
echo "$(date '+%Y-%m-%d %H:%M:%S') WARNING: High memory usage detected" >> $ERROR_LOG

if [ -f "$ACCESS_LOG" ]; then
    echo "=== Access Log Summary ==="
    echo "Total requests: $(wc -l < $ACCESS_LOG)"
    echo "Unique IPs: $(awk '{print $2}' $ACCESS_LOG | sort -u | wc -l)"
    echo "Status code distribution:"
    awk '{print $5}' $ACCESS_LOG | sort | uniq -c | sort -nr
    echo
fi

if [ -f "$ERROR_LOG" ]; then
    echo "=== Error Log Summary ==="
    echo "Total errors: $(wc -l < $ERROR_LOG)"
    echo "Error types:"
    grep -o "ERROR\|WARNING\|INFO" $ERROR_LOG | sort | uniq -c
    echo
fi

echo "=== System Status ==="
echo "Uptime: $(uptime)"
echo "Disk usage: $(df -h / | tail -1 | awk '{print $5}')"
echo "Memory usage: $(free -h | grep Mem | awk '{printf "%.1f%%", $3/$2 * 100.0}')"
EOF

sudo chmod +x /opt/webapp/bin/analyze_logs.sh
sudo chown webapp:webapp /opt/webapp/bin/analyze_logs.sh

# Test the scripts
sudo -u webapp /opt/webapp/bin/monitor.sh
sudo -u webapp /opt/webapp/bin/analyze_logs.sh
```

---

## 📚 Additional Resources

### Essential System Administration Resources

**Books**:
- **"UNIX and Linux System Administration Handbook"**: Comprehensive reference
- **"The Linux Programming Interface"**: Deep dive into Linux internals
- **"Linux Security Cookbook"**: Security-focused administration

**Online Resources**:
- **Linux Documentation Project**: Comprehensive guides and HOWTOs
- **Red Hat System Administrator's Guide**: Excellent even for non-Red Hat systems
- **Ubuntu Server Guide**: Great for Ubuntu-specific administration

**Practice Platforms**:
- **OverTheWire Bandit**: Security-focused Linux challenges
- **Linux Academy**: Hands-on labs and scenarios
- **Katacoda**: Interactive Linux scenarios

### Command Reference Sheets

**Permission Quick Reference**:
```
755 = rwxr-xr-x = Owner: all, Group: read+execute, Others: read+execute
644 = rw-r--r-- = Owner: read+write, Group: read, Others: read
600 = rw------- = Owner: read+write, Group: none, Others: none
777 = rwxrwxrwx = Everyone: all permissions (DANGEROUS!)
```

**Process Signal Reference**:
```
SIGTERM (15) = Graceful shutdown
SIGKILL (9)  = Force kill (cannot be caught)
SIGHUP (1)   = Hang up (reload configuration)
SIGSTOP (19) = Pause process
SIGCONT (18) = Resume process
```

---

## 🔄 Day 3 Assessment

### Permission Mastery Check (10 minutes)

**Challenge 1**: Set up a shared directory where:
- Owner (alice) can read, write, execute
- Group (developers) can read and execute
- Others have no access

<details>
<summary>Click for solution</summary>

```bash
mkdir shared_project
chown alice:developers shared_project
chmod 750 shared_project
# or
chmod u=rwx,g=rx,o= shared_project
```
</details>

**Challenge 2**: Find all files in `/var/log` that are writable by group or others (potential security issue):

<details>
<summary>Click for solution</summary>

```bash
find /var/log -type f \( -perm -020 -o -perm -002 \) -ls
```
</details>

### Process Management Challenge (10 minutes)

**Scenario**: A Python script is consuming 90% CPU. Write the commands to:
1. Identify the process
2. Get detailed information about it
3. Gracefully stop it
4. Verify it's stopped

<details>
<summary>Sample Solution</summary>

```bash
# 1. Identify the process
ps aux --sort=-%cpu | head -5
# or
top -n 1 | grep python

# 2. Get detailed information
ps -ef | grep python
lsof -p [PID]

# 3. Gracefully stop it
kill -TERM [PID]
# Wait a few seconds, then check if it's still running
ps -p [PID]

# 4. If still running, force kill
kill -KILL [PID]

# Verify it's stopped
ps -p [PID]
# Should return "No such process"
```
</details>

### Text Processing Challenge (10 minutes)

**Task**: Given a log file with this format:
```
2024-01-15 10:30:15 192.168.1.100 GET /api/users 200 1024
2024-01-15 10:30:16 192.168.1.101 POST /api/login 404 512
```

Write a one-liner to find the top 5 IP addresses by request count:

<details>
<summary>Click for solution</summary>

```bash
awk '{print $3}' logfile.txt | sort | uniq -c | sort -nr | head -5
```
</details>

---

## 🚀 Tomorrow's Preview: Networking Basics for DevOps

Tomorrow we'll explore the networking fundamentals that every DevOps engineer needs:

- **TCP/IP fundamentals** (how the internet actually works)
- **DNS and domain management** (making names work)
- **Load balancing concepts** (distributing traffic)
- **Firewalls and security groups** (controlling access)
- **Network troubleshooting** (when things go wrong)
- **Container networking** (Docker and Kubernetes networking)

**Preparation for Tomorrow**:
- Practice today's commands until they're second nature
- Try to solve a real problem using file permissions
- Monitor your own system's processes and resources
- Think about how networking affects the applications you use daily

---

## 🎉 Congratulations!

You've completed Day 3 and gained crucial system administration skills! You now know:
- ✅ Linux file permissions and security model
- ✅ Process management and system monitoring
- ✅ Package management and software installation
- ✅ Environment variable configuration
- ✅ Advanced text processing with grep, sed, and awk
- ✅ Real-world troubleshooting techniques

**Key Takeaway**: System administration is about understanding how all the pieces fit together. Permissions, processes, and monitoring are the foundation of reliable systems. Master these, and you can troubleshoot almost any Linux issue.

**Tomorrow**: We'll dive into networking - the glue that connects all systems together!

---

*"In Linux, everything is a file, and every file tells a story. Learn to read the stories, and you'll understand the system."* - Ready to connect the world? 🌐

### **[🎯 Continue to Day 4: Networking Basics for DevOps →](day04.md)**
echo "$(date '+%