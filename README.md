# Azure Security Setup Project

![Azure](https://img.shields.io/badge/Microsoft_Azure-0078D4?style=for-the-badge&logo=microsoft-azure&logoColor=white)
![Security](https://img.shields.io/badge/Security-Enabled-green?style=for-the-badge)
![Linux](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![Status](https://img.shields.io/badge/Status-Active-success?style=for-the-badge)

## 🎯 Project Overview

This project demonstrates the implementation of fundamental cloud security practices in Microsoft Azure. I built a secure cloud environment from scratch, focusing on network security, monitoring, and incident detection. The project showcases essential skills for cloud security engineering and DevSecOps roles.

**Key Achievement:** Created a production-grade security configuration that successfully blocked unauthorized access attempts while maintaining monitoring and alerting capabilities for security events.

---

## 🏗️ Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    Azure Cloud Environment                   │
│                                                               │
│  ┌────────────────────────────────────────────────────────┐ │
│  │           Resource Group: rg-security-project          │ │
│  │                                                          │ │
│  │  ┌──────────────────────────────────────────────────┐  │ │
│  │  │     Virtual Network: vnet-security-lab           │  │ │
│  │  │          Address Space: 10.0.0.0/16              │  │ │
│  │  │                                                    │  │ │
│  │  │  ┌────────────────────────────────────────────┐  │  │ │
│  │  │  │  Subnet: subnet-default (10.0.1.0/24)      │  │ │ │
│  │  │  │                                              │  │ │ │
│  │  │  │  ┌──────────────────────────────────────┐  │  │ │ │
│  │  │  │  │  Network Security Group (NSG)        │  │ │ │ │
│  │  │  │  │  ┌────────────────────────────────┐  │  │ │ │ │
│  │  │  │  │  │  Rule 1: Allow SSH (Port 22)   │  │ │ │ │ │
│  │  │  │  │  │  Source: My IP Only            │  │ │ │ │ │
│  │  │  │  │  │  Priority: 100                 │  │ │ │ │ │
│  │  │  │  │  └────────────────────────────────┘  │  │ │ │ │
│  │  │  │  │  ┌────────────────────────────────┐  │  │ │ │ │
│  │  │  │  │  │  Rule 2: Allow HTTPS (443)     │  │ │ │ │ │
│  │  │  │  │  │  Source: Any                   │  │ │ │ │ │
│  │  │  │  │  │  Priority: 110                 │  │ │ │ │ │
│  │  │  │  │  └────────────────────────────────┘  │  │ │ │ │
│  │  │  │  │  ┌────────────────────────────────┐  │  │ │ │ │
│  │  │  │  │  │  Default: Deny All Inbound     │  │ │ │ │ │
│  │  │  │  │  │  Priority: 65000               │  │ │ │ │ │
│  │  │  │  │  └────────────────────────────────┘  │  │ │ │ │
│  │  │  │  └──────────────────────────────────────┘  │ │ │ │
│  │  │  │                    │                        │ │ │ │
│  │  │  │         ┌──────────▼──────────┐            │ │ │ │
│  │  │  │         │  Virtual Machine    │            │ │ │ │
│  │  │  │         │  vm-security-lab    │            │ │ │ │
│  │  │  │         │  Ubuntu 24.04 LTS   │            │ │ │ │
│  │  │  │         │  Size: B1s          │            │ │ │ │
│  │  │  │         └─────────────────────┘            │ │ │ │
│  │  │  └────────────────────────────────────────────┘ │ │ │
│  │  └──────────────────────────────────────────────────┘ │ │
│  │                                                         │ │
│  │  ┌──────────────────────────────────────────────────┐ │ │
│  │  │     Log Analytics Workspace                       │ │ │
│  │  │     law-security-monitor                          │ │ │
│  │  │     • Performance Metrics                         │ │ │
│  │  │     • Security Events                             │ │ │
│  │  │     • System Logs                                 │ │ │
│  │  └──────────────────────────────────────────────────┘ │ │
│  │                           │                            │ │
│  │                           ▼                            │ │
│  │  ┌──────────────────────────────────────────────────┐ │ │
│  │  │     Azure Monitor Alerts                          │ │ │
│  │  │     • High CPU Usage Alert (>80%)                 │ │ │
│  │  │     • Failed SSH Login Attempts                   │ │ │
│  │  │     • Email Notifications                         │ │ │
│  │  └──────────────────────────────────────────────────┘ │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │  Authorized IP  │
                  │  (My Location)  │
                  └─────────────────┘
```

---

## 🛠️ Technologies Used

| Technology | Purpose |
|------------|---------|
| **Microsoft Azure** | Cloud platform for hosting infrastructure |
| **Azure Virtual Machines** | Linux VM running Ubuntu 24.04 LTS |
| **Network Security Groups (NSG)** | Firewall rules for network-level security |
| **Azure Monitor** | Performance and health monitoring |
| **Log Analytics** | Centralized logging and query engine |
| **KQL (Kusto Query Language)** | Log analysis and security event detection |
| **Azure Alerts** | Automated incident notification system |
| **SSH** | Secure remote access protocol |

---

## ✨ Key Features Implemented

### 1. **Network Security**
- ✅ Configured Network Security Group with least-privilege access
- ✅ Implemented IP-based access control (whitelist only)
- ✅ Blocked all unauthorized inbound traffic by default
- ✅ Allowed only essential ports (SSH: 22, HTTPS: 443)

### 2. **Monitoring & Observability**
- ✅ Enabled Azure Monitor with VM Insights
- ✅ Configured Log Analytics workspace for centralized logging
- ✅ Implemented performance metric collection (CPU, memory, disk, network)
- ✅ Set up boot diagnostics for troubleshooting

### 3. **Alerting & Incident Response**
- ✅ Created high CPU usage alert (threshold: 80%)
- ✅ Configured failed SSH authentication monitoring
- ✅ Set up email notifications for security events
- ✅ Established action groups for incident response

### 4. **Security Best Practices**
- ✅ Enabled auto-shutdown to minimize attack surface and costs
- ✅ Implemented SSH key-based authentication
- ✅ Regular system updates and patching strategy
- ✅ Principle of least privilege for all configurations

---

## 🔐 Security Configurations

### Network Security Group Rules

| Priority | Name | Port | Protocol | Source | Action | Purpose |
|----------|------|------|----------|--------|--------|---------|
| 100 | Allow-SSH-MyIP | 22 | TCP | My IP | Allow | Secure administrative access |
| 110 | Allow-HTTPS | 443 | TCP | Any | Allow | Web traffic for updates |
| 65000 | DenyAllInbound | * | * | Any | Deny | Default deny rule |

**Security Rationale:**
- **SSH limited to my IP**: Prevents brute-force attacks from unauthorized sources
- **HTTPS allowed**: Enables package updates and secure web communication
- **Default deny**: Following zero-trust principles, all other traffic is blocked

### Authentication & Access
- **Method**: SSH public key authentication
- **Backup**: Password authentication via Azure Reset Password feature
- **Access Control**: IP-based whitelisting in NSG
- **Emergency Access**: Azure Serial Console and Cloud Shell

---

## 📊 Monitoring and Alerts Setup

### Metrics Collected
- CPU Percentage (every 60 seconds)
- Available Memory
- Disk I/O operations
- Network in/out
- System uptime and heartbeat

### Alert Rules Configured

**Alert 1: High CPU Usage**
- **Condition**: CPU > 80% for 5 minutes
- **Severity**: Warning (Level 2)
- **Action**: Email notification to admin
- **Evaluation Frequency**: Every 5 minutes
- **Purpose**: Detect potential DoS attacks or resource abuse

**Alert 2: Failed SSH Login Attempts**
- **Condition**: Multiple authentication failures detected
- **Severity**: Error (Level 1)
- **Action**: Immediate email notification
- **Query**: Syslog authentication failures
- **Purpose**: Detect potential brute-force attacks

### Log Analytics Queries Used

```kql
// Query 1: Failed SSH Authentication Attempts
Syslog
| where Computer contains "vm-security-lab"
| where Facility == "auth" or Facility == "authpriv"
| where SeverityLevel == "err"
| project TimeGenerated, Computer, ProcessName, SyslogMessage
| order by TimeGenerated desc

// Query 2: CPU Performance Trend
Perf
| where Computer contains "vm-security-lab"
| where CounterName == "% Processor Time"
| summarize avg(CounterValue) by bin(TimeGenerated, 5m)
| render timechart

// Query 3: VM Heartbeat Check
Heartbeat
| where Computer contains "vm-security-lab"
| summarize LastHeartbeat = max(TimeGenerated) by Computer
```

---

## 🧪 Testing & Validation

### Security Testing Results

#### NSG Rule Validation
- ✅ **Test 1**: Successfully connected from authorized IP (203.x.x.x)
- ❌ **Test 2**: Connection blocked from VPN IP (104.x.x.x) - **Expected behavior**
- ✅ **Test 3**: HTTPS traffic allowed for system updates
- **Conclusion**: NSG rules functioning correctly, unauthorized access prevented

#### Monitoring Validation
- ✅ CPU stress test triggered alert within 8 minutes
- ✅ Alert email received with correct severity and details
- ✅ Peak CPU usage reached 98% during test
- **Conclusion**: Monitoring and alerting operational and responsive

#### Logging Validation
- ✅ Successfully queried VM logs in Log Analytics
- ✅ Captured 5 failed SSH authentication attempts (testing)
- ✅ Performance metrics collected consistently
- ✅ Logs retained for 30 days (configurable)
- **Conclusion**: Comprehensive logging enabled and queryable

---

## 📚 What I Learned

### Technical Skills
1. **Azure Infrastructure**: Hands-on experience deploying and configuring VMs, VNets, and NSGs
2. **Network Security**: Understanding of firewall rules, port management, and access control
3. **Cloud Monitoring**: Implementation of observability using Azure Monitor and Log Analytics
4. **KQL Proficiency**: Writing queries to detect security events and analyze performance
5. **Incident Response**: Creating actionable alerts for security and performance issues

### Security Concepts
- **Defense in Depth**: Multiple layers of security (NSG, authentication, monitoring)
- **Least Privilege Principle**: Granting only necessary access
- **Zero Trust Model**: Deny by default, allow by exception
- **Security Monitoring**: Importance of continuous monitoring and alerting
- **Incident Detection**: Proactive threat detection through log analysis

### Best Practices
- Always test security controls after implementation
- Document configurations and rationale for audit purposes
- Implement cost controls (auto-shutdown) in non-production environments
- Regular review of NSG rules and access patterns
- Balance security with operational efficiency

---

## 🚀 Future Improvements

### Short-term Enhancements
- [ ] Implement Azure Bastion for more secure remote access
- [ ] Add Azure Security Center recommendations
- [ ] Configure diagnostic settings for NSG flow logs
- [ ] Set up automated backup solution
- [ ] Implement patch management automation

### Medium-term Goals
- [ ] Deploy Azure Key Vault for secret management
- [ ] Add Azure DDoS Protection
- [ ] Implement Just-in-Time (JIT) VM access
- [ ] Configure Microsoft Defender for Cloud
- [ ] Set up geo-redundant backup

### Long-term Vision
- [ ] Multi-region deployment with load balancing
- [ ] Implement Infrastructure as Code (Terraform/Bicep)
- [ ] CI/CD pipeline for configuration updates
- [ ] SIEM integration for advanced threat detection
- [ ] Compliance automation (CIS benchmarks)

---

## 🔄 How to Replicate This Project

### Prerequisites
- Azure account (free tier available with $200 credit)
- Basic understanding of networking concepts
- SSH client (built-in on Mac/Linux, PuTTY for Windows)
- 2-3 hours of time

### Step-by-Step Guide

#### Phase 1: Environment Setup (30 minutes)
1. Create Azure account and activate subscription
2. Create Resource Group: `rg-security-project`
3. Deploy Virtual Network: `vnet-security-lab` (10.0.0.0/16)
4. Create subnet: `subnet-default` (10.0.1.0/24)

#### Phase 2: Security Configuration (45 minutes)
5. Create Network Security Group: `nsg-vm-security`
6. Configure NSG rules:
   - Allow SSH from your IP (port 22)
   - Allow HTTPS (port 443)
   - Verify default deny rule
7. Deploy Ubuntu VM (B1s size recommended)
8. Associate NSG with VM network interface

#### Phase 3: Monitoring Setup (45 minutes)
9. Create Log Analytics workspace: `law-security-monitor`
10. Enable Azure Monitor on the VM
11. Configure diagnostic settings
12. Create action group for email alerts
13. Set up CPU usage alert (>80%)
14. Configure failed login alert

#### Phase 4: Testing & Validation (30 minutes)
15. Test NSG rules with authorized/unauthorized IPs
16. Run CPU stress test to trigger alert
17. Attempt failed SSH logins to test logging
18. Verify all alerts and logs working
19. Document findings with screenshots

#### Phase 5: Documentation (30 minutes)
20. Take screenshots of all configurations
21. Export NSG rules and alert configurations
22. Create architecture diagram
23. Write project summary and learnings

### Estimated Costs
- **VM (B1s)**: ~$10-15/month (or free with credits)
- **Log Analytics**: First 5GB/month free
- **Storage**: Minimal (~$1/month)
- **Total**: Can be run entirely within free tier credits

### Clean-up Instructions
```bash
# Delete entire resource group (deletes all resources)
az group delete --name rg-security-project --yes --no-wait
```

---

## 📁 Repository Structure

```
azure-security-setup-project/
├── README.md                          # This file
├── docs/
│   ├── architecture-diagram.png       # Visual architecture
│   ├── setup-guide.md                 # Detailed setup instructions
│   └── testing-results.md             # Test documentation
├── screenshots/
│   ├── nsg-rules.png                  # NSG configuration
│   ├── vm-overview.png                # VM dashboard
│   ├── monitoring-alerts.png          # Alert configurations
│   ├── log-analytics-query.png        # Sample KQL queries
│   └── alert-email.png                # Alert notification
├── scripts/
│   ├── nsg-rules.json                 # Exported NSG configuration
│   ├── alert-queries.kql              # Log Analytics queries
│   └── stress-test.sh                 # CPU stress test script
├── configs/
│   └── alert-rules.json               # Alert rule definitions
└── LICENSE                            # MIT License
```

---

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🤝 Connect With Me

- **LinkedIn**: [Your LinkedIn Profile]
- **GitHub**: [Your GitHub Profile]
- **Portfolio**: [Your Portfolio Website]
- **Email**: [Your Email]

---

## 🙏 Acknowledgments

- Microsoft Azure documentation and learning paths
- Azure Security Best Practices guides
- Cloud security community for knowledge sharing

---

## 📊 Project Stats

![Last Updated](https://img.shields.io/badge/Last%20Updated-October%202025-blue)
![Maintenance](https://img.shields.io/badge/Maintained-Yes-green)
![Azure](https://img.shields.io/badge/Azure-Tested-success)

---

**⭐ If you found this project helpful, please consider giving it a star!**

