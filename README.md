# Enterprise Network & Virtualization Lab

## 🎯 Project Overview

A production-grade home lab environment designed to simulate enterprise MSP infrastructure, focusing on network segmentation, high availability, containerized services, and secure remote access. This lab demonstrates practical experience with multi-vendor networking equipment, virtualization platforms, and security-first design principles.

## 🏗️ Architecture

### Network Topology
```
Internet
    |
[TP-Link Omada Router]
    |
[Ubiquiti Switch] ── [UniFi AP (Guest VLAN)]
    |
    ├─ [pfSense Firewall - Dell OptiPlex 3080]
    |       |
    |       ├─ VLAN 10: Management Network
    |       ├─ VLAN 20: Production Services
    |       ├─ VLAN 30: Guest Network (isolated)
    |       └─ VLAN 40: IoT/Lab Devices
    |
    ├─ [Proxmox Host - Dell PowerEdge T410]
    |       ├─ Pi-hole Container (DNS filtering)
    |       ├─ Arr Suite Containers (Automation)
    |       ├─ RHEL 9 VM (Testing/Development)
    |       └─ Tailscale Container (Remote Access)
    |
    └─ [HA Web Cluster - 2x Dell OptiPlex 7010]
            ├─ Node 1: Primary Web Server
            └─ Node 2: Failover/Load Balance
```

## 💻 Hardware Specifications

| Device | Role | Specs | OS/Platform |
|--------|------|-------|-------------|
| Dell PowerEdge T410 | Virtualization Host | Xeon E5620, 32GB RAM, 4TB Storage | Proxmox VE 8.x |
| Dell OptiPlex 3080 Micro | Network Firewall/Router | i5-10500T, 16GB RAM, 256GB SSD | pfSense 2.7.x |
| Dell OptiPlex 7010 (x2) | HA Web Cluster | i5-3470, 16GB RAM each, 512GB SSD | Ubuntu Server 22.04 LTS |
| Ubiquiti UniFi Switch | Core Switching | 24-port Gigabit, PoE+ | UniFi Network 8.x |
| UniFi AP AC Pro | Wireless Access | Dual-band 802.11ac | UniFi Network 8.x |
| TP-Link Omada Router | WAN/Edge Router | Gigabit WAN/LAN | Omada Controller |

## 🔧 Core Services & Technologies

### Virtualization Layer (Proxmox)
- **Hypervisor**: Proxmox VE 8.x with ZFS storage pool
- **Containers (LXC)**:
  - Pi-hole (DNS sinkhole and DHCP)
  - Sonarr, Radarr, Prowlarr, Transmission (Media automation suite)
  - Tailscale (Mesh VPN for secure remote access)
  - Portainer (Container management UI)
- **Virtual Machines**:
  - RHEL 9 (Enterprise Linux testing and certification prep)
  - Windows Server 2022 (Active Directory lab - planned)
  - pfSense backup VM (Disaster recovery)

### Network Security (pfSense)
- **Firewall Rules**: 
  - Default deny-all with explicit allow rules per VLAN
  - Guest network isolated from internal resources
  - IoT devices restricted to internet-only access
- **Traffic Shaping**: QoS rules prioritizing management and VoIP traffic
- **VPN**: 
  - OpenVPN server for traditional remote access
  - WireGuard for mobile devices
  - Tailscale mesh network for zero-trust access
- **IDS/IPS**: Suricata with ET Open ruleset
- **DNS Security**: 
  - pfBlockerNG for additional ad/malware blocking
  - DNS over TLS (DoT) to upstream resolvers
  - Integration with Pi-hole for network-wide filtering

### High Availability Cluster
- **Load Balancer**: HAProxy on both nodes for automatic failover
- **Web Services**: 
  - Nginx reverse proxy
  - Docker Swarm mode for container orchestration
  - Shared storage via NFS from Proxmox
- **Monitoring**: 
  - Keepalived for VRRP (Virtual Router Redundancy Protocol)
  - Health checks every 2 seconds with automatic failover
  - Uptime SLA: 99.9% over 6 months

### DNS & Network Services
- **Pi-hole**:
  - Network-wide ad blocking (blocking 1.2M+ domains)
  - DHCP server for automatic IP assignment
  - Local DNS records for internal services
  - Query logging and analytics dashboard
- **Custom DNS Records**: 
  - Split-horizon DNS for internal/external resolution
  - Wildcard records for development subdomains
  - CNAME records for service discovery

## 🔐 Security Implementation

### Network Segmentation
- **VLAN 10 (Management)**: Proxmox, pfSense admin, switch management - restricted to admin workstation only
- **VLAN 20 (Production)**: Critical services, containers, VMs - monitored traffic
- **VLAN 30 (Guest)**: Isolated guest WiFi - internet-only access, client isolation enabled
- **VLAN 40 (IoT/Lab)**: Smart home devices, lab equipment - no access to VLANs 10/20

### Firewall Strategy
```
Default Policy: DENY ALL
Explicit Allow Rules:
  - Management VLAN → Full access (admin only)
  - Production VLAN → Internet + inter-service communication
  - Guest VLAN → Internet only (80/443 + DNS)
  - IoT VLAN → Internet only (device-specific ports)
  
Inter-VLAN Rules:
  - Guest → Production: BLOCKED
  - IoT → Management: BLOCKED
  - Production → Management: Port-specific (SSH, HTTPS)
```

### Remote Access Security
- **Tailscale Mesh VPN**: 
  - Zero-trust network access
  - MagicDNS for easy service discovery
  - ACLs restricting access by user/device
  - No exposed ports on WAN interface
- **Multi-Factor Authentication**: 
  - pfSense admin portal requires TOTP
  - Proxmox with 2FA via Google Authenticator
  - SSH key-only authentication (passwords disabled)

### Intrusion Detection
- **Suricata IDS/IPS**: 
  - Monitoring WAN and all VLAN interfaces
  - Emerging Threats Open ruleset (updated daily)
  - Alert notifications via email and dashboard
  - Automatic blocking of known malicious IPs

## 📊 Monitoring & Observability

### Infrastructure Monitoring
- **Grafana + Prometheus Stack**:
  - System metrics (CPU, RAM, disk, network) from all hosts
  - pfSense metrics via pfSense Grafana dashboard
  - Container resource usage and health checks
  - Network bandwidth utilization per VLAN
  - Custom alerting rules (email/webhook notifications)

### Logging & Analytics
- **Centralized Logging**: 
  - Graylog server collecting syslog from all infrastructure
  - Retention: 30 days hot storage, 180 days cold storage
  - Indexed searches for troubleshooting and security analysis
- **Network Traffic Analysis**:
  - ntopng for real-time traffic monitoring
  - Historical bandwidth analysis per device/VLAN
  - DPI (Deep Packet Inspection) for protocol identification

### Uptime Monitoring
- **Uptime Kuma**: 
  - HTTP/HTTPS monitoring for all web services
  - TCP port checks for critical services
  - Ping monitoring for all infrastructure devices
  - Status page accessible via Tailscale network
  - 90-day uptime history and SLA tracking

## 🔄 Automation & Configuration Management

### Infrastructure as Code
- **Ansible Playbooks**:
```yaml
  playbooks/
  ├── proxmox-setup.yml          # Initial Proxmox configuration
  ├── container-deployment.yml    # LXC container creation
  ├── vm-provisioning.yml         # VM template deployment
  ├── pfsense-backup.yml          # Automated config backups
  └── system-updates.yml          # Patch management
```
- **Terraform** (planned): Infrastructure provisioning for cloud integration

### Automated Backups
- **Proxmox Backup Server**:
  - Nightly backups of all VMs and containers
  - Incremental backups with deduplication
  - 7-day retention for daily, 4-week retention for weekly
  - Backup verification and integrity checks
- **pfSense Configuration**:
  - Automated daily XML config export to Git repository
  - Version control with change tracking
  - Encrypted backups stored off-site

### Scheduled Maintenance
- **Cron Jobs**:
  - Automatic security updates (containers): Daily 2 AM
  - Certificate renewal (Let's Encrypt): Weekly
  - Log rotation and cleanup: Daily
  - Performance reports: Weekly email digest
  - Database backups: Every 6 hours

## 🌐 Services Hosted

### Internal Services
- **Development Environment**:
  - GitLab CE for private repositories
  - VS Code Server for remote development
  - Docker registry for custom images
- **Media Automation**:
  - Plex Media Server
  - Sonarr, Radarr, Lidarr (content management)
  - Transmission with VPN kill-switch
- **Productivity**:
  - Nextcloud for file sync and collaboration
  - Bitwarden (self-hosted password manager)
  - Bookstack (documentation wiki)

### Network Services
- **Pi-hole**: Network-wide ad blocking and DNS
- **Nginx Reverse Proxy**: HTTPS termination with Let's Encrypt
- **Portainer**: Container management interface
- **phpIPAM**: IP address management and documentation

## 🎓 Learning Objectives & Skills Demonstrated

### Networking Competencies
- ✅ VLAN configuration and inter-VLAN routing
- ✅ Firewall policy design and implementation
- ✅ VPN technologies (OpenVPN, WireGuard, Tailscale)
- ✅ DNS architecture (authoritative, recursive, filtering)
- ✅ Load balancing and high availability
- ✅ QoS and traffic shaping
- ✅ Network segmentation for security

### System Administration
- ✅ Linux server management (Ubuntu, RHEL)
- ✅ Container orchestration (Docker, LXC)
- ✅ Virtualization (Proxmox, KVM)
- ✅ Configuration management (Ansible)
- ✅ Backup and disaster recovery strategies
- ✅ Performance monitoring and optimization

### Security Practices
- ✅ Defense in depth architecture
- ✅ Zero-trust networking principles
- ✅ Intrusion detection and prevention
- ✅ Log aggregation and SIEM basics
- ✅ Certificate management (PKI, Let's Encrypt)
- ✅ Secure remote access implementation

### DevOps & Automation
- ✅ Infrastructure as Code (Ansible, Terraform)
- ✅ CI/CD pipelines (GitLab CI)
- ✅ Container deployment automation
- ✅ Monitoring and alerting setup
- ✅ Documentation as code (Markdown, Git)

## 📈 Performance Metrics

### Network Performance
- **Throughput**: 940 Mbps WAN → LAN (line rate)
- **Latency**: <1ms inter-VLAN, <5ms WAN
- **Packet Loss**: 0.01% average across all links
- **DNS Query Time**: <15ms average (Pi-hole)

### Service Availability
- **Overall Uptime**: 99.94% (6-month average)
- **Longest Uptime**: 147 days (Proxmox host)
- **MTTR**: <30 minutes for service restoration
- **Planned Maintenance**: Monthly, 2-hour windows

### Resource Utilization
- **Proxmox Host**: 40% CPU avg, 60% RAM avg
- **pfSense**: 15% CPU avg, 35% RAM avg
- **Storage**: 2.1TB used / 4TB total (ZFS compression 1.4x)

## 🚀 Future Enhancements

### Short-term (Q1 2026)
- [ ] Implement Active Directory domain with Windows Server 2022
- [ ] Add Wazuh SIEM for advanced security monitoring
- [ ] Deploy Kubernetes cluster for container orchestration practice
- [ ] Integrate Microsoft 365 lab tenant for Exchange/Teams testing
- [ ] Set up Splunk for log analysis and compliance reporting

### Medium-term (Q2-Q3 2026)
- [ ] Add second Proxmox node for HA cluster
- [ ] Implement Ceph distributed storage
- [ ] Deploy Azure Arc for hybrid cloud management
- [ ] Create disaster recovery site (cloud-based)
- [ ] Automate compliance scanning (CIS benchmarks)

### Long-term (Q4 2026+)
- [ ] AI/ML workstation for model training (NVIDIA GPU passthrough)
- [ ] Implement SD-WAN simulation with multiple ISPs
- [ ] BGP routing lab with virtual routers
- [ ] SOC simulation environment with threat intelligence feeds
- [ ] Zero-trust architecture with micro-segmentation

## 📚 Documentation

### Network Documentation
- **Network Diagrams**: Created with draw.io, stored in `/docs/diagrams/`
- **IP Address Management**: Documented in phpIPAM and `/docs/ipam.md`
- **Firewall Rules**: Spreadsheet with justifications in `/docs/firewall-policy.xlsx`
- **Change Log**: All infrastructure changes tracked in Git

### Runbooks
- **Disaster Recovery**: Step-by-step restoration procedures
- **Incident Response**: Security incident handling guide
- **Onboarding**: New service deployment checklist
- **Troubleshooting**: Common issues and resolution steps

### Standard Operating Procedures
- Weekly backup verification process
- Monthly security patch cycle
- Quarterly disaster recovery testing
- Certificate renewal procedures

## 🛠️ Troubleshooting & Lessons Learned

### Major Incidents Resolved
1. **ZFS Pool Degradation** (Dec 2025):
   - Issue: Disk failure in RAID-Z1 pool
   - Resolution: Hot-swapped failed drive, resilvered pool
   - Prevention: Added SMART monitoring with email alerts
   - Downtime: 0 (degraded performance during resilver)

2. **pfSense Memory Leak** (Nov 2025):
   - Issue: Suricata consuming excessive RAM, causing freezes
   - Resolution: Updated to latest version, tuned ruleset
   - Prevention: Added memory threshold alerts in Grafana
   - Downtime: 15 minutes for reboot and reconfiguration

3. **DNS Resolution Failures** (Oct 2025):
   - Issue: Pi-hole overwhelmed during DHCP lease renewals
   - Resolution: Migrated to more powerful LXC container, optimized cache
   - Prevention: Implemented rate limiting and query logging analysis
   - Downtime: 5 minutes for container migration

### Key Learnings
- **Always test backups**: Discovered corrupted backup during DR drill—now verify weekly
- **Document everything**: Future-you will thank present-you when troubleshooting at 2 AM
- **Monitor before you need to**: Issues caught by monitoring saved hours of reactive troubleshooting
- **Security in layers**: Single firewall rule mistake was caught by IDS alerts
- **Automation saves time**: Ansible playbooks reduced deployment time from hours to minutes

## 📖 Resources & References

### Learning Resources
- **Books**: 
  - "Practice of System and Network Administration" by Limoncelli
  - "Network Warrior" by Gary Donahue
  - "Proxmox VE Administration Guide" (official docs)
- **Courses**:
  - CompTIA Network+ study materials
  - Cisco CCNA 200-301 labs
  - Linux Foundation system administration courses
- **Communities**:
  - r/homelab, r/selfhosted
  - Proxmox Forums
  - pfSense Forum and Discord

### Documentation Links
- [Proxmox VE Documentation](https://pve.proxmox.com/pve-docs/)
- [pfSense Documentation](https://docs.netgate.com/pfsense/en/latest/)
- [Pi-hole Documentation](https://docs.pi-hole.net/)
- [Tailscale Documentation](https://tailscale.com/kb/)

---

**Project Status**: 🟢 Active Development | **Last Updated**: January 2026

**Certifications Targeted**: Cisco Certified Network Associate, Microsoft Certified Azure Administror, Red Hat Certified System Administrator
