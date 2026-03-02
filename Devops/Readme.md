
# 📚 DevOps & Linux Interview Preparation Index

> **Recommended Study Order:** Start with Sections 1–3 (Linux + Networking), then Sections 4–5 (Containers + Kubernetes), then Section 6 (AWS), then Sections 7–10. Section 11 should be practiced throughout.

---

## 🐧 Section 1: Linux Fundamentals

### 1.1 Linux Architecture & Boot Process
- Kernel, shell, and user space
- BIOS → GRUB → Kernel → init/systemd boot sequence
- Runlevels vs systemd targets

### 1.2 Process Management
- `fork()`, `exec()`, process lifecycle
- Process states (R, S, D, Z, T)
- Signals (SIGTERM, SIGKILL, SIGHUP, SIGCHLD)
- Zombie and orphan processes
- `nice`, `renice`, process priority
- `nohup`, `screen`, `tmux`

### 1.3 File System & Storage
- Linux directory structure (`/etc`, `/var`, `/proc`, `/sys`, `/dev`)
- Inodes, hard links, soft links
- File permissions (rwx, SUID, SGID, sticky bit)
- Disk management: `df`, `du`, `lsblk`, `fdisk`, `parted`
- LVM (Logical Volume Manager)
- Mount points, `/etc/fstab`
- Filesystem types: ext4, xfs, tmpfs

### 1.4 Users & Groups
- `/etc/passwd`, `/etc/shadow`, `/etc/group`
- `useradd`, `usermod`, `userdel`
- `sudo`, `su`, sudoers file
- PAM (Pluggable Authentication Modules)

### 1.5 Package Management
- `apt` / `apt-get` (Debian/Ubuntu)
- `yum` / `dnf` (RHEL/Amazon Linux)
- `rpm`, `dpkg` internals
- Package repositories

### 1.6 Shell Scripting
- Bash scripting fundamentals
- Variables, loops, conditionals, functions
- Exit codes and error handling
- Cron jobs and scheduling
- `sed`, `awk`, `grep`, `cut`, `sort`, `uniq`
- `xargs`, `find`, pipes and redirection

---

## 🔧 Section 2: System Administration

### 2.1 Systemd & Service Management
- Unit files (`.service`, `.timer`, `.socket`, `.target`)
- `systemctl` commands
- `journald` and `journalctl`
- Service dependencies (`Requires`, `Wants`, `After`)
- Restart policies and failure handling

### 2.2 Memory Management
- Virtual memory, page tables, page cache
- `free`, `vmstat`, `/proc/meminfo`
- OOM Killer (`oom_score`, `oom_score_adj`)
- Swap management and swappiness
- Memory leak detection

### 2.3 CPU & Performance
- Load average explained
- CPU scheduling, context switching
- `top`, `htop`, `mpstat`, `sar`
- `perf`, `strace`, `ltrace`
- Identifying CPU bottlenecks

### 2.4 Disk I/O & Performance
- `iostat`, `iotop`, `blktrace`
- I/O schedulers
- Read/write throughput vs IOPS
- Disk bottleneck identification

### 2.5 Logging & Monitoring
- `syslog`, `rsyslog`, `journald`
- `/var/log` structure
- Log rotation (`logrotate`)
- `dmesg`, `auditd`
- Linux performance tools (USE method)

### 2.6 Security Hardening
- SSH hardening (key-based auth, disable root login)
- `iptables` / `nftables` / `firewalld`
- SELinux / AppArmor basics
- File integrity monitoring
- CIS benchmarks

---

## 🌐 Section 3: Networking

### 3.1 Networking Fundamentals
- OSI model and TCP/IP stack
- IP addressing, subnetting, CIDR
- TCP vs UDP, three-way handshake
- DNS resolution (`/etc/hosts`, `/etc/resolv.conf`, `nsswitch.conf`)
- HTTP/HTTPS, TLS/SSL basics

### 3.2 Linux Networking Tools
- `ip`, `ss`, `netstat`, `tcpdump`, `wireshark`
- `ping`, `traceroute`, `mtr`, `nmap`
- `curl`, `wget`, `nc` (netcat)
- `iptables` rules and NAT

### 3.3 TCP Connection States
- TIME_WAIT, CLOSE_WAIT, ESTABLISHED
- Connection pooling
- Ephemeral port exhaustion

### 3.4 Load Balancing & Proxies
- Layer 4 vs Layer 7 load balancing
- nginx as reverse proxy and load balancer
- HAProxy
- Health checks and failover

### 3.5 Network Namespaces & Virtual Networking
- veth pairs, bridges, tun/tap
- How container networking works
- iptables and NAT in containers

---

## 🐳 Section 4: Containers & Docker

### 4.1 Container Fundamentals
- Containers vs VMs
- Linux namespaces (pid, net, mnt, uts, ipc, user)
- cgroups (resource limits)
- Union filesystems (overlay2)

### 4.2 Docker
- Docker architecture (daemon, client, registry)
- Dockerfile best practices
- Multi-stage builds
- Image layers and caching
- Docker networking (bridge, host, overlay, none)
- Docker volumes and bind mounts
- `docker-compose`
- Container security (non-root user, read-only FS, capabilities)

### 4.3 Container Registry
- ECR (Elastic Container Registry)
- Image scanning and vulnerability management
- Image tagging strategies

---

## ☸️ Section 5: Kubernetes (EKS Focus)

### 5.1 Kubernetes Architecture
- Control plane (API server, etcd, scheduler, controller manager)
- Worker nodes (kubelet, kube-proxy, container runtime)
- etcd — role and importance

### 5.2 Core Kubernetes Objects
- Pod, ReplicaSet, Deployment
- StatefulSet, DaemonSet, Job, CronJob
- Service (ClusterIP, NodePort, LoadBalancer, ExternalName)
- Ingress and IngressController
- ConfigMap and Secret
- Namespace

### 5.3 Kubernetes Networking
- Pod-to-pod communication (CNI plugins: VPC CNI, Calico, Cilium)
- Service discovery (kube-dns / CoreDNS)
- Network policies
- Ingress controllers (ALB Ingress Controller, nginx)

### 5.4 Kubernetes Storage
- PersistentVolume (PV) and PersistentVolumeClaim (PVC)
- StorageClass and dynamic provisioning
- EBS CSI driver, EFS CSI driver

### 5.5 Kubernetes Scheduling & Resource Management
- Resource requests and limits
- LimitRange and ResourceQuota
- Node affinity, pod affinity, taints and tolerations
- Horizontal Pod Autoscaler (HPA)
- Vertical Pod Autoscaler (VPA)
- Cluster Autoscaler / Karpenter

### 5.6 Kubernetes Security
- RBAC (Role, ClusterRole, RoleBinding)
- ServiceAccounts and IRSA (IAM Roles for Service Accounts)
- Pod Security Standards / Pod Security Admission
- Network policies
- Secrets management (AWS Secrets Manager + External Secrets Operator)
- OPA / Gatekeeper

### 5.7 EKS-Specific Topics
- EKS managed vs self-managed node groups
- Fargate profiles
- EKS add-ons (CoreDNS, kube-proxy, VPC CNI, EBS CSI)
- EKS upgrade strategy
- EKS networking (VPC CNI, secondary CIDR)
- EKS authentication (aws-auth ConfigMap, EKS access entries)

### 5.8 Kubernetes Observability
- `kubectl` debugging commands
- Liveness, readiness, startup probes
- Container logs (`kubectl logs`)
- Events and describe
- Metrics server, Prometheus + Grafana
- Fluentd / Fluent Bit for log aggregation

### 5.9 Helm
- Helm charts structure
- `values.yaml`, templates, helpers
- Helm release lifecycle
- Helm hooks

---

## ☁️ Section 6: AWS Core Services

### 6.1 Compute
- EC2 (instance types, AMIs, user data, metadata service)
- Auto Scaling Groups (launch templates, scaling policies)
- Lambda (serverless, cold start, concurrency)

### 6.2 Networking
- VPC (subnets, route tables, IGW, NAT Gateway)
- Security Groups vs NACLs
- VPC Peering, Transit Gateway, PrivateLink
- Route 53 (DNS, routing policies, health checks)
- CloudFront (CDN, origins, behaviors)
- ELB (ALB, NLB, CLB — differences and use cases)

### 6.3 Storage
- S3 (storage classes, lifecycle policies, versioning, replication)
- EBS (volume types: gp3, io2, st1, sc1)
- EFS (NFS, performance modes)
- Glacier

### 6.4 IAM & Security
- IAM users, roles, policies, groups
- Trust policies and resource-based policies
- STS and AssumeRole
- AWS Organizations and SCPs
- KMS (key management, envelope encryption)
- Secrets Manager vs Parameter Store
- GuardDuty, Security Hub, Config, CloudTrail

### 6.5 Databases
- RDS (Multi-AZ, Read Replicas, parameter groups)
- Aurora (cluster architecture, serverless)
- DynamoDB (partition key, GSI, LSI, DAX)
- ElastiCache (Redis vs Memcached)

### 6.6 Messaging & Eventing
- SQS (standard vs FIFO, DLQ, visibility timeout)
- SNS (topics, subscriptions, fan-out pattern)
- EventBridge
- Kinesis (streams, firehose, analytics)

### 6.7 Monitoring & Observability
- CloudWatch (metrics, logs, alarms, dashboards)
- CloudWatch Logs Insights
- X-Ray (distributed tracing)
- AWS Config (compliance, config rules)
- CloudTrail (API audit logging)

---

## 🔄 Section 7: CI/CD & DevOps Practices

### 7.1 Git & Version Control
- Git internals (objects, refs, HEAD)
- Branching strategies (GitFlow, trunk-based)
- Merge vs rebase
- Git hooks
- Monorepo vs polyrepo

### 7.2 CI/CD Pipelines
- CI/CD concepts and pipeline stages
- Jenkins (pipelines, agents, shared libraries)
- GitHub Actions (workflows, actions, runners)
- AWS CodePipeline, CodeBuild, CodeDeploy
- GitLab CI/CD
- ArgoCD (GitOps, sync policies, app of apps)
- Flux CD

### 7.3 Deployment Strategies
- Rolling update
- Blue/Green deployment
- Canary deployment
- Feature flags
- Rollback strategies

### 7.4 Infrastructure as Code
- Terraform (providers, state, modules, workspaces)
- Terraform state management (remote state, locking)
- Terraform best practices
- AWS CDK (constructs, stacks, environments)
- CloudFormation (stacks, nested stacks, StackSets, drift detection)
- Pulumi basics

### 7.5 Configuration Management
- Ansible (playbooks, roles, inventory, modules)
- Ansible best practices for AWS
- Chef / Puppet basics

---

## 📊 Section 8: Observability & SRE

### 8.1 The Three Pillars of Observability
- Metrics (what is happening)
- Logs (why it happened)
- Traces (where it happened)

### 8.2 Prometheus & Grafana
- Prometheus architecture (scraping, exporters, alertmanager)
- PromQL basics
- Grafana dashboards
- Recording rules and alerting rules

### 8.3 Log Management
- ELK Stack (Elasticsearch, Logstash, Kibana)
- OpenSearch (AWS managed)
- Fluentd / Fluent Bit
- Loki + Grafana

### 8.4 Distributed Tracing
- OpenTelemetry
- AWS X-Ray
- Jaeger / Zipkin

### 8.5 SRE Concepts
- SLI, SLO, SLA definitions
- Error budgets
- Toil reduction
- Incident management and postmortems
- Chaos engineering basics

---

## 🔐 Section 9: Security & Compliance (DevSecOps)

### 9.1 DevSecOps Principles
- Shift-left security
- SAST, DAST, SCA tools
- Container image scanning (Trivy, Snyk, ECR scanning)
- Secret scanning in pipelines

### 9.2 Kubernetes Security
- CIS Kubernetes Benchmark
- `kube-bench`
- Falco (runtime security)
- OPA/Gatekeeper policies

### 9.3 AWS Security
- Well-Architected Framework — Security Pillar
- AWS Security Hub findings
- GuardDuty threat detection
- Compliance frameworks (SOC2, PCI-DSS, HIPAA on AWS)

---

## 🏗️ Section 10: Architecture & Design

### 10.1 High Availability & Fault Tolerance
- Multi-AZ vs Multi-Region
- RTO and RPO
- Active-Active vs Active-Passive
- Circuit breaker pattern

### 10.2 Scalability Patterns
- Horizontal vs vertical scaling
- Auto-scaling strategies
- Caching strategies (CDN, application cache, database cache)
- Database sharding and read replicas

### 10.3 Microservices & Service Mesh
- Microservices vs monolith trade-offs
- Service mesh (Istio, AWS App Mesh)
- API Gateway patterns
- Event-driven architecture

### 10.4 Cost Optimization
- AWS Cost Explorer, Budgets, Savings Plans
- Right-sizing instances
- Spot instances and interruption handling
- S3 intelligent tiering

---

## 🎯 Section 11: Interview-Specific Preparation

### 11.1 Behavioral Questions (STAR Method)
- Leadership Principles alignment
- Conflict resolution scenarios
- Ownership and bias for action examples
- Customer obsession stories

### 11.2 System Design Questions
- Design a CI/CD pipeline for a microservices app
- Design a highly available 3-tier application on AWS
- Design a Kubernetes-based deployment platform
- Design a centralized logging solution

### 11.3 Troubleshooting Scenarios
- "Production is down" — systematic debugging approach
- High CPU / memory / disk scenarios
- Network connectivity issues
- Kubernetes pod not starting

### 11.4 Coding & Scripting
- Bash scripting challenges
- Python for automation (boto3, subprocess)
- Terraform coding exercises
- Kubernetes manifest writing
