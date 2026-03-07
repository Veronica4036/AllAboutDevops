# Day 4: Networking Basics for DevOps 🌐

*"How Systems Communicate - The Foundation of Modern Infrastructure"*

**Estimated Time**: 3 hours  
**Difficulty**: Beginner to Intermediate  
**Prerequisites**: Day 3 completed, basic understanding of Linux commands

---

## 🎯 Learning Objectives

By the end of today, you will:
- Understand TCP/IP fundamentals and how the internet works
- Master DNS concepts and domain management
- Configure and troubleshoot network connectivity
- Understand load balancing and traffic distribution
- Implement basic firewall rules and security groups
- Troubleshoot network issues like a pro
- **Interview Ready**: Confidently discuss networking in technical interviews

---

## 📖 Theory Section: How the Internet Really Works

### The Networking Stack: Your Digital Postal System

Imagine sending a letter across the world. You need:
- **Your address** (source IP)
- **Destination address** (destination IP)
- **Postal service** (routers and switches)
- **Delivery method** (TCP/UDP protocols)
- **Letter format** (HTTP, HTTPS, etc.)

This is exactly how computer networking works! Let's break it down:

### The OSI Model: The Seven Layers of Communication

Think of the OSI model like a multi-story building where each floor has a specific job:

```
7. Application Layer    📱 (HTTP, HTTPS, FTP, SSH)
6. Presentation Layer   🔒 (Encryption, Compression)
5. Session Layer        🤝 (Session Management)
4. Transport Layer      📦 (TCP, UDP - Reliable Delivery)
3. Network Layer        🗺️ (IP - Routing and Addressing)
2. Data Link Layer      🔗 (Ethernet, WiFi - Local Delivery)
1. Physical Layer       ⚡ (Cables, Radio Waves)
```

**Real-World Example**: When you visit `https://google.com`:
1. **Application Layer**: Your browser creates an HTTP request
2. **Presentation Layer**: HTTPS encrypts the data
3. **Session Layer**: Establishes a session with Google's server
4. **Transport Layer**: TCP ensures reliable delivery
5. **Network Layer**: IP routes packets across the internet
6. **Data Link Layer**: Ethernet/WiFi delivers packets locally
7. **Physical Layer**: Electrical signals travel through cables

### 💡 Did You Know?

**Netflix's Network Engineering**: Netflix delivers 15% of the world's internet traffic! They use a technique called "Open Connect" where they place servers directly inside internet service providers' networks. This reduces latency and ensures smooth streaming. Their network engineers are some of the highest-paid in the industry because they solve problems at massive scale.

---

## 🌍 IP Addressing: The Internet's Address System

### IPv4: The Current Standard

IPv4 addresses are like postal addresses for computers:
```
192.168.1.100
│   │   │ │
│   │   │ └── Host (specific device)
│   │   └── Subnet (local network)
│   └── Network (organization)
└── Network (region/ISP)
```

**Common IP Address Ranges**:
- **Private Networks** (not routable on internet):
  - `10.0.0.0/8` (10.0.0.0 to 10.255.255.255)
  - `172.16.0.0/12` (172.16.0.0 to 172.31.255.255)
  - `192.168.0.0/16` (192.168.0.0 to 192.168.255.255)
- **Public Networks**: Everything else (routable on internet)
- **Loopback**: `127.0.0.1` (localhost - your own machine)

### Subnetting: Dividing Networks

Think of subnetting like dividing a city into neighborhoods:

```bash
# Network: 192.168.1.0/24
# Subnet mask: 255.255.255.0
# Available hosts: 192.168.1.1 to 192.168.1.254
# Network address: 192.168.1.0
# Broadcast address: 192.168.1.255
```

**CIDR Notation**:
- `/24` = 255.255.255.0 (256 addresses, 254 usable)
- `/16` = 255.255.0.0 (65,536 addresses)
- `/8` = 255.0.0.0 (16,777,216 addresses)

### IPv6: The Future (Already Here!)

IPv6 addresses look like: `2001:0db8:85a3:0000:0000:8a2e:0370:7334`

**Why IPv6 Matters**:
- IPv4 has only 4.3 billion addresses (we've run out!)
- IPv6 has 340 undecillion addresses (that's 340 with 36 zeros!)
- Better security and performance
- Required for modern cloud deployments

---

## 🔧 Essential Networking Commands

### Network Configuration Commands

**`ip` - Modern Network Configuration**:
```bash
# Show all network interfaces
ip addr show
# or shorter
ip a

# Show specific interface
ip addr show eth0

# Show routing table
ip route show

# Add IP address to interface
sudo ip addr add 192.168.1.100/24 dev eth0

# Bring interface up/down
sudo ip link set eth0 up
sudo ip link set eth0 down
```

**`ifconfig` - Legacy Network Configuration** (still widely used):
```bash
# Show all interfaces
ifconfig

# Show specific interface
ifconfig eth0

# Configure IP address
sudo ifconfig eth0 192.168.1.100 netmask 255.255.255.0

# Bring interface up/down
sudo ifconfig eth0 up
sudo ifconfig eth0 down
```

### Network Testing Commands

**`ping` - Test Connectivity**:
```bash
# Basic ping
ping google.com

# Ping specific number of times
ping -c 4 google.com

# Ping with specific interval
ping -i 2 google.com

# Ping IPv6
ping6 google.com
```

**`traceroute` - Trace Network Path**:
```bash
# Trace route to destination
traceroute google.com

# Trace route with IP addresses only
traceroute -n google.com

# Trace route using TCP instead of ICMP
traceroute -T google.com
```

**`nslookup` and `dig` - DNS Lookup**:
```bash
# Basic DNS lookup
nslookup google.com

# Detailed DNS information
dig google.com

# Query specific record type
dig google.com MX
dig google.com AAAA

# Reverse DNS lookup
dig -x 8.8.8.8

# Query specific DNS server
dig @8.8.8.8 google.com
```

### Network Monitoring Commands

**`netstat` - Network Statistics**:
```bash
# Show all connections
netstat -a

# Show listening ports
netstat -l

# Show TCP connections with process info
netstat -tulpn

# Show network statistics
netstat -s

# Show routing table
netstat -r
```

**`ss` - Socket Statistics** (modern replacement for netstat):
```bash
# Show all sockets
ss -a

# Show listening TCP sockets
ss -tl

# Show UDP sockets
ss -u

# Show processes using sockets
ss -p

# Show sockets on specific port
ss -tulpn | grep :80
```

**`lsof` - List Open Files** (including network connections):
```bash
# Show network connections
lsof -i

# Show connections on specific port
lsof -i :80

# Show connections by specific process
lsof -p 1234

# Show IPv4 connections only
lsof -i4

# Show IPv6 connections only
lsof -i6
```

---

## 🏗️ DNS: The Internet's Phone Book

### How DNS Works: The Name Resolution Process

When you type `www.example.com` in your browser:

1. **Browser Cache**: Check if IP is already known
2. **OS Cache**: Check system DNS cache
3. **Router Cache**: Check local router cache
4. **ISP DNS**: Query Internet Service Provider's DNS server
5. **Root Servers**: Query root DNS servers (13 worldwide)
6. **TLD Servers**: Query Top Level Domain servers (.com, .org, etc.)
7. **Authoritative Servers**: Query domain's authoritative DNS servers
8. **Response**: IP address returned back through the chain

### DNS Record Types

**A Record** - Maps domain to IPv4 address:
```
www.example.com.    IN    A    192.168.1.100
```

**AAAA Record** - Maps domain to IPv6 address:
```
www.example.com.    IN    AAAA    2001:db8::1
```

**CNAME Record** - Creates an alias:
```
blog.example.com.    IN    CNAME    www.example.com.
```

**MX Record** - Mail exchange servers:
```
example.com.    IN    MX    10    mail.example.com.
```

**TXT Record** - Text information (often used for verification):
```
example.com.    IN    TXT    "v=spf1 include:_spf.google.com ~all"
```

**NS Record** - Name servers:
```
example.com.    IN    NS    ns1.example.com.
```

### DNS Configuration and Troubleshooting

**Configure DNS Servers**:
```bash
# Edit DNS configuration
sudo nano /etc/resolv.conf

# Add DNS servers
nameserver 8.8.8.8
nameserver 8.8.4.4
nameserver 1.1.1.1
```

**DNS Troubleshooting Commands**:
```bash
# Test DNS resolution
nslookup example.com

# Detailed DNS query
dig example.com +trace

# Check all DNS records
dig example.com ANY

# Test specific DNS server
dig @8.8.8.8 example.com

# Reverse DNS lookup
dig -x 192.168.1.100
```

### Real-World DNS Example: Setting Up a Web Server

```bash
# Check current DNS settings
cat /etc/resolv.conf

# Test DNS resolution
dig mywebsite.com

# Check if domain points to correct IP
dig mywebsite.com A

# Verify mail servers
dig mywebsite.com MX

# Check DNS propagation
dig @8.8.8.8 mywebsite.com
dig @1.1.1.1 mywebsite.com
```

---

## ⚖️ Load Balancing: Distributing the Load

### What is Load Balancing?

Imagine a popular restaurant with one waiter versus ten waiters. Load balancing is like having multiple waiters to serve customers efficiently:

- **Single Server**: One waiter serves all customers (slow, single point of failure)
- **Load Balanced**: Multiple waiters share the workload (fast, resilient)

### Load Balancing Algorithms

**Round Robin**: Requests distributed evenly
```
Request 1 → Server A
Request 2 → Server B
Request 3 → Server C
Request 4 → Server A (cycle repeats)
```

**Least Connections**: Route to server with fewest active connections
```
Server A: 5 connections
Server B: 3 connections ← Next request goes here
Server C: 7 connections
```

**Weighted Round Robin**: Servers get different amounts of traffic
```
Server A (weight 3): Gets 3 requests
Server B (weight 2): Gets 2 requests
Server C (weight 1): Gets 1 request
```

**IP Hash**: Route based on client IP
```
hash(client_ip) % number_of_servers = target_server
```

### Types of Load Balancers

**Layer 4 (Transport Layer)**:
- Routes based on IP and port
- Faster, less CPU intensive
- Cannot inspect application data

**Layer 7 (Application Layer)**:
- Routes based on HTTP headers, URLs, cookies
- More intelligent routing
- Can perform SSL termination

### Simple Load Balancer with Nginx

```nginx
# /etc/nginx/nginx.conf
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;
}

server {
    listen 80;
    server_name myapp.com;
    
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## 🔥 Firewalls and Security Groups

### Understanding Firewalls: Digital Bouncers

Firewalls are like security guards at a nightclub:
- **Allow**: Known good traffic (VIP list)
- **Deny**: Known bad traffic (banned list)
- **Inspect**: Unknown traffic (check ID)

### iptables: Linux Firewall

**Basic iptables Commands**:
```bash
# List current rules
sudo iptables -L

# Allow SSH (port 22)
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP (port 80)
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Allow HTTPS (port 443)
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Block specific IP
sudo iptables -A INPUT -s 192.168.1.100 -j DROP

# Allow traffic from specific subnet
sudo iptables -A INPUT -s 192.168.1.0/24 -j ACCEPT

# Save rules (Ubuntu/Debian)
sudo iptables-save > /etc/iptables/rules.v4
```

**Common Firewall Scenarios**:
```bash
# Web server firewall rules
sudo iptables -A INPUT -i lo -j ACCEPT  # Allow loopback
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT   # SSH
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT   # HTTP
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT  # HTTPS
sudo iptables -A INPUT -j DROP  # Drop everything else

# Database server (only allow app servers)
sudo iptables -A INPUT -s 192.168.1.10 -p tcp --dport 3306 -j ACCEPT
sudo iptables -A INPUT -s 192.168.1.11 -p tcp --dport 3306 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 3306 -j DROP
```

### UFW: Uncomplicated Firewall

**UFW Commands** (easier than iptables):
```bash
# Enable UFW
sudo ufw enable

# Allow SSH
sudo ufw allow ssh
# or
sudo ufw allow 22

# Allow HTTP and HTTPS
sudo ufw allow 'Apache Full'
# or
sudo ufw allow 80
sudo ufw allow 443

# Allow from specific IP
sudo ufw allow from 192.168.1.100

# Allow specific port from specific IP
sudo ufw allow from 192.168.1.100 to any port 3306

# Deny traffic
sudo ufw deny 23

# Show status
sudo ufw status verbose

# Delete rule
sudo ufw delete allow 80
```

### Cloud Security Groups (AWS Example)

Security groups are like virtual firewalls for cloud instances:

```bash
# Create security group
aws ec2 create-security-group \
    --group-name web-server-sg \
    --description "Security group for web servers"

# Allow HTTP traffic
aws ec2 authorize-security-group-ingress \
    --group-name web-server-sg \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0

# Allow HTTPS traffic
aws ec2 authorize-security-group-ingress \
    --group-name web-server-sg \
    --protocol tcp \
    --port 443 \
    --cidr 0.0.0.0/0

# Allow SSH from specific IP
aws ec2 authorize-security-group-ingress \
    --group-name web-server-sg \
    --protocol tcp \
    --port 22 \
    --cidr 203.0.113.0/24
```

---

## 🔍 Network Troubleshooting: Detective Work

### The Systematic Approach to Network Issues

When network problems occur, follow this methodology:

1. **Define the Problem**: What exactly isn't working?
2. **Gather Information**: When did it start? What changed?
3. **Test Connectivity**: Can you reach the destination?
4. **Check DNS**: Is name resolution working?
5. **Verify Routing**: Is traffic taking the right path?
6. **Examine Firewalls**: Are security rules blocking traffic?
7. **Monitor Performance**: Is it slow or completely broken?

### Common Network Troubleshooting Scenarios

**Scenario 1: "I can't reach the website"**
```bash
# Step 1: Test basic connectivity
ping google.com

# Step 2: Test DNS resolution
nslookup google.com
dig google.com

# Step 3: Test with IP address directly
ping 8.8.8.8

# Step 4: Check local network configuration
ip addr show
ip route show

# Step 5: Test specific port
telnet google.com 80
# or
nc -zv google.com 80
```

**Scenario 2: "The application is slow"**
```bash
# Test latency
ping -c 10 server.com

# Trace network path
traceroute server.com

# Check for packet loss
ping -c 100 server.com | grep loss

# Test bandwidth
iperf3 -c server.com

# Check local network utilization
iftop
# or
nethogs
```

**Scenario 3: "SSH connection refused"**
```bash
# Test if port is open
telnet server.com 22
# or
nc -zv server.com 22

# Check if SSH service is running
ssh -v user@server.com

# Check firewall rules
sudo iptables -L | grep 22
sudo ufw status

# Check SSH configuration
sudo sshd -T
```

### Network Monitoring Tools

**`tcpdump` - Packet Capture**:
```bash
# Capture packets on interface
sudo tcpdump -i eth0

# Capture HTTP traffic
sudo tcpdump -i eth0 port 80

# Capture traffic to/from specific host
sudo tcpdump -i eth0 host google.com

# Save capture to file
sudo tcpdump -i eth0 -w capture.pcap

# Read from file
tcpdump -r capture.pcap
```

**`wireshark` - GUI Packet Analyzer**:
```bash
# Install Wireshark
sudo apt install wireshark

# Run Wireshark (GUI)
sudo wireshark

# Command-line version
tshark -i eth0
```

**`iftop` - Interface Traffic Monitor**:
```bash
# Monitor network traffic by connection
sudo iftop

# Monitor specific interface
sudo iftop -i eth0

# Show port numbers
sudo iftop -P
```

---

## 🎯 Interview Preparation: Networking Scenarios

### Question 1: Network Connectivity Troubleshooting

**Interviewer**: "A user reports they can't access a web application. Walk me through your troubleshooting process."

**How to Think Through This**:
1. Start with the basics (connectivity)
2. Work through the network stack systematically
3. Consider both client and server-side issues
4. Document findings for future reference

**Sample Answer**:
```bash
# Step 1: Verify basic connectivity
ping webapp.company.com

# Step 2: Test DNS resolution
nslookup webapp.company.com
dig webapp.company.com

# Step 3: Test specific service port
telnet webapp.company.com 80
nc -zv webapp.company.com 443

# Step 4: Check local network configuration
ip addr show
ip route show
cat /etc/resolv.conf

# Step 5: Test from different location
# Try from another machine/network

# Step 6: Check server-side
ssh admin@webapp.company.com
sudo netstat -tulpn | grep :80
sudo systemctl status apache2
```

**Follow-up Explanation**: "I follow a systematic approach starting with basic connectivity and working up the network stack. I test both name resolution and direct IP access to isolate DNS issues. I also verify the service is actually running and listening on the expected port."

### Question 2: DNS Resolution Issues

**Interviewer**: "Users report that www.company.com sometimes resolves to the wrong IP address. How would you investigate?"

**Sample Answer**:
```bash
# Check current DNS resolution
dig www.company.com
nslookup www.company.com

# Test multiple DNS servers
dig @8.8.8.8 www.company.com
dig @1.1.1.1 www.company.com
dig @208.67.222.222 www.company.com

# Check DNS propagation
dig www.company.com +trace

# Verify authoritative servers
dig www.company.com NS
dig @ns1.company.com www.company.com

# Check TTL values
dig www.company.com | grep -i ttl

# Test from different locations
# Use online DNS checker tools

# Check local DNS cache
sudo systemctl flush-dns
# or
sudo systemd-resolve --flush-caches
```

**Explanation**: "DNS issues often involve caching or propagation delays. I test multiple DNS servers to see if results are consistent. The +trace option shows the complete resolution path, helping identify where the problem occurs. TTL values tell us how long records are cached."

### Question 3: Load Balancer Configuration

**Interviewer**: "You need to set up a load balancer for a web application with 3 servers. One server is more powerful than the others. How would you configure it?"

**Sample Answer**:
```nginx
# Nginx load balancer configuration
upstream backend {
    # Powerful server gets more traffic (weight 3)
    server 192.168.1.10:8080 weight=3;
    
    # Standard servers get normal traffic (weight 1)
    server 192.168.1.11:8080 weight=1;
    server 192.168.1.12:8080 weight=1;
    
    # Health checks
    server 192.168.1.10:8080 max_fails=3 fail_timeout=30s;
    server 192.168.1.11:8080 max_fails=3 fail_timeout=30s;
    server 192.168.1.12:8080 max_fails=3 fail_timeout=30s;
}

server {
    listen 80;
    server_name myapp.com;
    
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        
        # Health check endpoint
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503;
    }
    
    # Health check endpoint
    location /health {
        access_log off;
        return 200 "healthy\n";
    }
}
```

**Explanation**: "I use weighted round-robin to give the powerful server 3x more traffic. Health checks ensure failed servers are automatically removed from rotation. The proxy headers preserve client information for the backend servers."

### Question 4: Firewall Security

**Interviewer**: "Design firewall rules for a 3-tier web application: web servers, application servers, and database servers."

**Sample Answer**:
```bash
# Web Server Firewall (DMZ)
# Allow internet traffic to web services
iptables -A INPUT -p tcp --dport 80 -j ACCEPT   # HTTP
iptables -A INPUT -p tcp --dport 443 -j ACCEPT  # HTTPS
iptables -A INPUT -p tcp --dport 22 -s 10.0.1.0/24 -j ACCEPT  # SSH from management

# Application Server Firewall (Private subnet)
# Only allow traffic from web servers
iptables -A INPUT -p tcp --dport 8080 -s 10.0.2.0/24 -j ACCEPT  # From web servers
iptables -A INPUT -p tcp --dport 22 -s 10.0.1.0/24 -j ACCEPT    # SSH from management

# Database Server Firewall (Private subnet)
# Only allow traffic from application servers
iptables -A INPUT -p tcp --dport 3306 -s 10.0.3.0/24 -j ACCEPT  # From app servers
iptables -A INPUT -p tcp --dport 22 -s 10.0.1.0/24 -j ACCEPT    # SSH from management

# Default deny for all tiers
iptables -A INPUT -j DROP
iptables -A FORWARD -j DROP
```

**Explanation**: "I implement defense in depth with network segmentation. Web servers are in a DMZ accepting internet traffic. Application servers only accept traffic from web servers. Database servers only accept traffic from application servers. Management access is restricted to specific subnets."

### Question 5: Performance Troubleshooting

**Interviewer**: "Users report that the application is slow during peak hours. How would you investigate network-related performance issues?"

**Sample Answer**:
```bash
# Check current network utilization
iftop -i eth0
nethogs  # Per-process network usage

# Test bandwidth to servers
iperf3 -c app-server.com

# Check for packet loss
ping -c 100 app-server.com | grep loss

# Monitor connection states
ss -s  # Summary of socket statistics
netstat -an | grep TIME_WAIT | wc -l  # Check for connection exhaustion

# Check network latency patterns
for i in {1..10}; do
    time curl -o /dev/null -s app-server.com
done

# Monitor DNS resolution time
dig app-server.com | grep "Query time"

# Check load balancer health
curl -I http://load-balancer.com/health

# Monitor server-side network
ssh app-server.com
top  # Check CPU usage
free -h  # Check memory
iostat 1 5  # Check I/O wait
```

**Explanation**: "Performance issues during peak hours often indicate capacity problems. I check both client and server-side metrics, focusing on bandwidth utilization, latency, and connection limits. The combination of tools helps identify whether the bottleneck is network, CPU, memory, or I/O related."

---

## 🎮 Hands-On Challenge: Network Infrastructure Setup

**Time**: 45 minutes  
**Scenario**: You're setting up network infrastructure for a new web application at "TechStart Inc."

### Mission 1: Network Configuration (15 minutes)

```bash
# Check current network configuration
ip addr show
ip route show

# Configure static IP (if needed)
sudo ip addr add 192.168.1.100/24 dev eth0
sudo ip route add default via 192.168.1.1

# Configure DNS
sudo tee /etc/resolv.conf << EOF
nameserver 8.8.8.8
nameserver 8.8.4.4
nameserver 1.1.1.1
EOF

# Test connectivity
ping -c 4 8.8.8.8
ping -c 4 google.com

# Test DNS resolution
nslookup google.com
dig google.com A
```

### Mission 2: Firewall Setup (15 minutes)

```bash
# Install and configure UFW
sudo apt update
sudo apt install ufw

# Set default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (be careful not to lock yourself out!)
sudo ufw allow ssh

# Allow web traffic
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Allow from specific management network
sudo ufw allow from 10.0.1.0/24

# Enable firewall
sudo ufw enable

# Check status
sudo ufw status verbose

# Test firewall rules
nc -zv localhost 22  # Should work
nc -zv localhost 80  # Should work
nc -zv localhost 25  # Should fail (SMTP not allowed)
```

### Mission 3: Network Monitoring Setup (15 minutes)

```bash
# Create network monitoring script
tee ~/network_monitor.sh << 'EOF'
#!/bin/bash

LOG_FILE="/var/log/network_monitor.log"
DATE=$(date '+%Y-%m-%d %H:%M:%S')

# Test connectivity to key services
GOOGLE_PING=$(ping -c 1 8.8.8.8 | grep "time=" | cut -d'=' -f4)
DNS_TIME=$(dig google.com | grep "Query time" | awk '{print $4}')

# Check network interface statistics
RX_BYTES=$(cat /sys/class/net/eth0/statistics/rx_bytes)
TX_BYTES=$(cat /sys/class/net/eth0/statistics/tx_bytes)

# Log results
echo "$DATE - Ping: ${GOOGLE_PING:-FAILED}, DNS: ${DNS_TIME:-FAILED}ms, RX: $RX_BYTES, TX: $TX_BYTES" >> $LOG_FILE

# Check for high network usage
CONNECTIONS=$(ss -tun | wc -l)
if [ $CONNECTIONS -gt 1000 ]; then
    echo "$DATE - WARNING: High connection count: $CONNECTIONS" >> $LOG_FILE
fi

# Test critical services
if ! nc -z localhost 80; then
    echo "$DATE - ERROR: Web server not responding" >> $LOG_FILE
fi

if ! nc -z localhost 22; then
    echo "$DATE - ERROR: SSH server not responding" >> $LOG_FILE
fi
EOF

chmod +x ~/network_monitor.sh

# Run the monitoring script
./network_monitor.sh

# View results
cat /var/log/network_monitor.log

# Set up cron job for regular monitoring
echo "*/5 * * * * /home/$(whoami)/network_monitor.sh" | crontab -
```

---

## 📚 Additional Resources

### Essential Networking Resources

**Books**:
- **"TCP/IP Illustrated" by W. Richard Stevens**: The definitive guide to TCP/IP
- **"Computer Networks" by Andrew Tanenbaum**: Comprehensive networking fundamentals
- **"Network Warrior" by Gary Donahue**: Practical networking for system administrators

**Online Resources**:
- **Cisco Networking Academy**: Free networking courses
- **Wireshark University**: Packet analysis tutorials
- **RFC Documents**: Official internet standards (start with RFC 791 for IP)

**Practice Platforms**:
- **GNS3**: Network simulation software
- **Packet Tracer**: Cisco's network simulation tool
- **EVE-NG**: Enterprise network emulation

### Network Troubleshooting Cheat Sheet

**Quick Connectivity Test**:
```bash
# Test in this order:
ping 127.0.0.1        # Loopback (local stack)
ping 192.168.1.1      # Default gateway
ping 8.8.8.8          # External IP (bypasses DNS)
ping google.com       # External hostname (tests DNS)
```

**Port Testing**:
```bash
# Test if port is open
nc -zv hostname port
telnet hostname port
nmap -p port hostname
```

**DNS Troubleshooting**:
```bash
# Test DNS resolution
nslookup hostname
dig hostname
host hostname

# Test specific DNS server
dig @8.8.8.8 hostname
```

---

## 🔄 Day 4 Assessment

### Network Fundamentals Check (10 minutes)

**Challenge 1**: Explain what happens when you type `www.google.com` in your browser and press Enter. Include all network layers.

<details>
<summary>Sample Answer</summary>

1. **Application Layer**: Browser creates HTTP request
2. **DNS Resolution**: Browser queries DNS to get IP address for www.google.com
3. **Transport Layer**: TCP connection established to port 80/443
4. **Network Layer**: IP packets routed through internet
5. **Data Link Layer**: Ethernet frames transmitted on local network
6. **Physical Layer**: Electrical signals sent over cables/wireless
7. **Response**: Google's server responds with HTML content
8. **Rendering**: Browser renders the webpage
</details>

**Challenge 2**: You can ping a server but can't SSH to it. List 5 possible causes.

<details>
<summary>Sample Answers</summary>

1. SSH service not running
1. SSH service not running on the server
2. Firewall blocking port 22
3. SSH configured to listen on different port
4. Authentication issues (wrong username/password/key)
5. SSH service configured to deny connections from your IP
6. Network ACLs blocking SSH traffic
7. SSH service crashed or misconfigured
</details>

### Practical Network Troubleshooting (15 minutes)

**Scenario**: A web application is intermittently slow. Users report it works fine sometimes but takes 30+ seconds to load other times.

**Your Investigation Plan Should Include**:
1. Network latency testing
2. DNS resolution time checks
3. Load balancer health verification
4. Server resource monitoring
5. Network path analysis

<details>
<summary>Sample Investigation</summary>

```bash
# Test network latency patterns
for i in {1..20}; do
    echo "Test $i: $(date)"
    time curl -o /dev/null -s http://webapp.com
    sleep 5
done

# Check DNS resolution consistency
for i in {1..10}; do
    dig webapp.com | grep "Query time"
done

# Test different DNS servers
dig @8.8.8.8 webapp.com
dig @1.1.1.1 webapp.com

# Check load balancer health
curl -I http://webapp.com/health

# Monitor network path
traceroute webapp.com

# Check for packet loss
ping -c 100 webapp.com | grep loss

# Test from different locations
# Use online tools or different servers
```
</details>

---

## 🚀 Tomorrow's Preview: Git Version Control Part 1

Tomorrow we'll dive into Git, the foundation of modern software development:

- **Git fundamentals** (understanding version control)
- **Repository management** (init, clone, remote)
- **Basic Git workflow** (add, commit, push, pull)
- **Branching strategies** (feature branches, merge vs rebase)
- **Collaboration workflows** (GitHub, GitLab, pull requests)
- **Git best practices** (commit messages, .gitignore, hooks)

**Preparation for Tomorrow**:
- Practice today's networking commands until they're natural
- Set up a simple network monitoring script
- Think about how version control helps in DevOps workflows
- Consider how Git enables collaboration in distributed teams

---

## 🎉 Congratulations!

You've completed Day 4 and gained essential networking knowledge! You now understand:
- ✅ TCP/IP fundamentals and how the internet works
- ✅ IP addressing, subnetting, and DNS resolution
- ✅ Network troubleshooting methodology and tools
- ✅ Load balancing concepts and configuration
- ✅ Firewall rules and security group management
- ✅ Network monitoring and performance analysis

**Key Takeaway**: Networking is the foundation that connects all systems in modern infrastructure. Understanding how data flows between systems helps you design resilient architectures and troubleshoot issues effectively. Every DevOps engineer needs solid networking fundamentals.

**Tomorrow**: We'll explore Git version control - the tool that enables collaboration and tracks every change in your infrastructure and applications!

---

*"The network is the computer. Master networking, and you master the foundation of all distributed systems."* - Ready to version control everything? 📝

### **[🎯 Continue to Day 5: Git Version Control Part 1 →](day05.md)**