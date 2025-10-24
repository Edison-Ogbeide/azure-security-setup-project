# Azure Security Setup - Detailed Implementation Guide

**Project**: Azure Security Setup Project  
**Author**: Endurance Joseph Ogbeide  
**Last Updated**: October 2025  
**Estimated Time**: 2-3 hours  
**Difficulty**: Beginner to Intermediate

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Environment Setup](#environment-setup)
3. [Network Configuration](#network-configuration)
4. [Virtual Machine Deployment](#virtual-machine-deployment)
5. [Security Hardening](#security-hardening)
6. [Monitoring Setup](#monitoring-setup)
7. [Alert Configuration](#alert-configuration)
8. [Testing & Validation](#testing--validation)
9. [Troubleshooting](#troubleshooting)
10. [Cost Management](#cost-management)

---

## Prerequisites

### Required Accounts & Tools

#### Azure Account
- [ ] Azure subscription (free tier available)
- [ ] $200 free credits for new accounts (30 days)
- [ ] Valid credit card for verification (won't be charged with free tier)

**Sign up**: https://azure.microsoft.com/free

#### Local Tools
- [ ] **SSH Client**:
  - Windows: Built-in (PowerShell), PuTTY, or Git Bash
  - Mac/Linux: Built-in Terminal
- [ ] **Web Browser**: Chrome, Edge, or Firefox (latest version)
- [ ] **Text Editor**: VS Code, Notepad++, or any editor for documentation

#### Knowledge Requirements
- [ ] Basic understanding of networking (IP addresses, ports, firewalls)
- [ ] Familiarity with command line/terminal
- [ ] Basic Linux commands (optional but helpful)
- [ ] Understanding of cloud computing concepts

### Pre-Setup Checklist

Before you begin, gather this information:

```
Your Public IP Address: _________________ (Google "what's my IP")
Preferred Azure Region: _________________ (e.g., East US, West Europe)
VM Username: ___________________ (e.g., azureuser)
VM Password: ___________________ (Strong: 12+ chars, mixed case, numbers, symbols)
Your Email for Alerts: _________________
```

---

## Environment Setup

### Step 1: Access Azure Portal

1. Navigate to https://portal.azure.com
2. Sign in with your Microsoft account
3. Accept terms and conditions if prompted
4. You should see the Azure Portal dashboard

**Expected Time**: 2 minutes

### Step 2: Create Resource Group

A Resource Group is a container that holds related Azure resources.

**Via Azure Portal:**

1. In the search bar at the top, type **"Resource Groups"**
2. Click **"Resource groups"** from results
3. Click **"+ Create"** button at the top

4. Fill in the details:
   ```
   Subscription: [Your subscription name]
   Resource group name: rg-security-project
   Region: East US (or nearest to you)
   ```

5. Click **"Review + create"**
6. Review the settings
7. Click **"Create"**
8. Wait for "Deployment complete" notification (~10 seconds)

**Via Azure CLI** (Alternative):
```bash
az group create \
  --name rg-security-project \
  --location eastus
```

**Verification:**
- Go to Resource Groups
- You should see `rg-security-project` listed
- Status should show "Succeeded"

**Expected Time**: 3 minutes

---

## Network Configuration

### Step 3: Create Virtual Network

A Virtual Network (VNet) provides isolated network space for your resources.

**Via Azure Portal:**

1. Search for **"Virtual networks"** in the top search bar
2. Click **"+ Create"**

3. **Basics tab**:
   ```
   Subscription: [Your subscription]
   Resource group: rg-security-project
   Name: vnet-security-lab
   Region: East US (must match resource group)
   ```

4. Click **"Next: IP Addresses"**

5. **IP Addresses tab**:
   ```
   IPv4 address space: 10.0.0.0/16
   
   Click "+ Add subnet"
   Subnet name: subnet-default
   Subnet address range: 10.0.1.0/24
   ```
   
   This gives you:
   - Total addresses: 65,536 (10.0.0.0 - 10.0.255.255)
   - Subnet addresses: 256 (10.0.1.0 - 10.0.1.255)
   - Usable host addresses: 251 (Azure reserves 5)

6. Click **"Review + create"** → **"Create"**

**Via Azure CLI** (Alternative):
```bash
# Create VNet
az network vnet create \
  --resource-group rg-security-project \
  --name vnet-security-lab \
  --address-prefix 10.0.0.0/16 \
  --subnet-name subnet-default \
  --subnet-prefix 10.0.1.0/24
```

**Verification:**
- Navigate to Virtual networks
- Open `vnet-security-lab`
- Check **"Subnets"** - should show `subnet-default`
- Check **"Connected devices"** - should be empty for now

**Expected Time**: 5 minutes

### Step 4: Create Network Security Group

Network Security Groups (NSG) are Azure's firewall rules.

**Via Azure Portal:**

1. Search for **"Network security groups"**
2. Click **"+ Create"**

3. Fill in details:
   ```
   Subscription: [Your subscription]
   Resource group: rg-security-project
   Name: nsg-vm-security
   Region: East US (must match resource group)
   ```

4. Click **"Review + create"** → **"Create"**

**Via Azure CLI** (Alternative):
```bash
az network nsg create \
  --resource-group rg-security-project \
  --name nsg-vm-security \
  --location eastus
```

**Verification:**
- Go to Network security groups
- Open `nsg-vm-security`
- Check **"Inbound security rules"** - should show default rules only

**Expected Time**: 3 minutes

### Step 5: Configure NSG Rules

Now we'll add custom firewall rules following the principle of least privilege.

**Understanding NSG Rule Properties:**
- **Priority**: 100-4096 (lower number = higher priority)
- **Source**: Who can initiate connection (IP, Service Tag, Any)
- **Destination**: Where traffic goes (usually "Any" for VM)
- **Port**: Which port to allow/deny
- **Action**: Allow or Deny

**Rule 1: Allow SSH from Your IP Only**

1. Open `nsg-vm-security`
2. Go to **"Inbound security rules"** (left sidebar)
3. Click **"+ Add"**

4. Configure:
   ```
   Source: IP Addresses
   Source IP addresses/CIDR ranges: [YOUR-PUBLIC-IP]/32
   Source port ranges: *
   Destination: Any
   Service: SSH
   Destination port ranges: 22
   Protocol: TCP
   Action: Allow
   Priority: 100
   Name: Allow-SSH-MyIP
   Description: Allow SSH access from my home/office IP only
   ```

5. Click **"Add"**

**Why `/32`?**: This means "exactly this IP address" - the most restrictive setting.

**Via Azure CLI** (Alternative):
```bash
az network nsg rule create \
  --resource-group rg-security-project \
  --nsg-name nsg-vm-security \
  --name Allow-SSH-MyIP \
  --priority 100 \
  --source-address-prefixes [YOUR-IP]/32 \
  --source-port-ranges '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 22 \
  --access Allow \
  --protocol Tcp \
  --description "Allow SSH from my IP only"
```

**Rule 2: Allow HTTPS (Optional but Recommended)**

This allows your VM to download updates securely.

1. Click **"+ Add"** again
2. Configure:
   ```
   Source: Any
   Source port ranges: *
   Destination: Any
   Service: HTTPS
   Destination port ranges: 443
   Protocol: TCP
   Action: Allow
   Priority: 110
   Name: Allow-HTTPS
   Description: Allow outbound HTTPS for updates and packages
   ```

3. Click **"Add"**

**Rule 3: Verify Default Deny Rule**

1. Scroll down in the inbound rules list
2. You should see a rule named **"DenyAllInbound"** with priority 65000
3. This catches all traffic not explicitly allowed above
4. **Do not modify or delete this rule**

**Final NSG Rules Summary:**

| Priority | Name | Port | Source | Action | Purpose |
|----------|------|------|--------|--------|---------|
| 100 | Allow-SSH-MyIP | 22 | Your IP | Allow | Admin access |
| 110 | Allow-HTTPS | 443 | Any | Allow | Updates |
| 65000 | DenyAllInbound | * | Any | Deny | Default deny |

**Security Note**: This configuration follows the "zero trust" model - deny everything by default, allow only what's necessary.

**Expected Time**: 10 minutes

---

## Virtual Machine Deployment

### Step 6: Deploy Ubuntu Virtual Machine

**Via Azure Portal:**

1. Search for **"Virtual machines"**
2. Click **"+ Create"** → **"Azure virtual machine"**

#### Basics Tab

```
Subscription: [Your subscription]
Resource group: rg-security-project
Virtual machine name: vm-security-lab
Region: East US (must match resource group)
Availability options: No infrastructure redundancy required
Security type: Standard
Image: Ubuntu Server 24.04 LTS - x64 Gen2
VM architecture: x64
Size: Standard_B1s (1 vcpu, 1 GiB memory) - Click "See all sizes" if not visible
```

**Authentication:**
```
Authentication type: SSH public key (Recommended)
   - Or choose "Password" if you prefer simplicity
Username: azureuser
SSH public key source: Generate new key pair
Key pair name: vm-security-lab_key
   
   OR if using Password:
   Username: azureuser
   Password: [Your strong password]
   Confirm password: [Same password]
```

**Inbound Port Rules:**
```
Public inbound ports: None (we'll use NSG instead)
```

Click **"Next: Disks"**

#### Disks Tab

```
OS disk type: Standard SSD (locally redundant storage)
Delete with VM: ✓ (checked)
Encryption type: (Default) Platform-managed key
```

Click **"Next: Networking"**

#### Networking Tab

**This is critical - pay attention!**

```
Virtual network: vnet-security-lab
Subnet: subnet-default (10.0.1.0/24)
Public IP: (new) vm-security-lab-ip
NIC network security group: Advanced
Configure network security group: nsg-vm-security (select your NSG)
Delete public IP and NIC when VM is deleted: ✓ (checked)
```

**Load balancing:**
```
Load balancing options: None
```

Click **"Next: Management"**

#### Management Tab

```
Identity:
  Enable system assigned managed identity: ☐ (unchecked for now)

Auto-shutdown:
  Enable auto-shutdown: ✓ (checked)
  Shutdown time: 7:00:00 PM (19:00)
  Time zone: (UTC) Coordinated Universal Time
  Notification before shutdown: ✓ (checked)
  Email address: [Your email]
  
Boot diagnostics:
  Enable boot diagnostics: ✓ (checked)
  
OS guest diagnostics:
  Enable OS guest diagnostics: ✓ (checked)
```

Click **"Next: Monitoring"**

#### Monitoring Tab

```
Alerts:
  Enable recommended alert rules: ☐ (we'll configure custom ones later)

Boot diagnostics:
  Already enabled in Management tab
```

Click **"Next: Advanced"** (skip this tab)

Click **"Next: Tags"** (optional)

#### Tags Tab (Optional but Recommended)

Tags help organize and track resources:

```
Name: Environment    Value: Development
Name: Project        Value: SecurityLab
Name: Owner          Value: [Your name]
Name: CostCenter     Value: Learning
```

Click **"Review + create"**

#### Review + Create

1. Review all settings carefully
2. Check estimated cost (should show ~$10-15/month)
3. If using SSH key, you'll see a notice: **"Generate new key pair"**
4. Click **"Create"**

**IMPORTANT**: If you chose SSH key authentication:
- A popup will appear: **"Generate new key pair"**
- Click **"Download private key and create resource"**
- Save the `.pem` file securely (you can't download it again!)
- Typical name: `vm-security-lab_key.pem`

**Deployment Progress:**
- You'll see "Deployment is in progress"
- This takes 3-5 minutes
- Wait for "Your deployment is complete"

**Expected Time**: 15 minutes

### Step 7: Verify VM Deployment

1. Click **"Go to resource"** after deployment completes
2. Check the VM overview page:
   ```
   Status: Running (green)
   Public IP address: [Note this down - e.g., 20.185.43.67]
   Private IP address: 10.0.1.4
   Operating system: Linux (Ubuntu 24.04)
   Size: Standard B1s
   ```

3. Note your **Public IP address** - you'll need this to connect

**Via Azure CLI** (Check deployment):
```bash
az vm list \
  --resource-group rg-security-project \
  --output table

az vm show \
  --resource-group rg-security-project \
  --name vm-security-lab \
  --show-details
```

**Expected Time**: 5 minutes

---

## Security Hardening

### Step 8: Connect to Your VM

Now let's verify connectivity and perform initial security setup.

#### For SSH Key Authentication (Linux/Mac):

```bash
# Set correct permissions on your key
chmod 400 ~/Downloads/vm-security-lab_key.pem

# Connect to VM
ssh -i ~/Downloads/vm-security-lab_key.pem azureuser@[YOUR-VM-PUBLIC-IP]

# Example:
ssh -i ~/Downloads/vm-security-lab_key.pem azureuser@20.185.43.67
```

#### For SSH Key Authentication (Windows PowerShell):

```powershell
# Navigate to Downloads folder
cd $HOME\Downloads

# Set permissions
icacls .\vm-security-lab_key.pem /inheritance:r
icacls .\vm-security-lab_key.pem /grant:r "$env:USERNAME:(R)"

# Connect to VM
ssh -i .\vm-security-lab_key.pem azureuser@[YOUR-VM-PUBLIC-IP]
```

#### For Password Authentication:

```bash
ssh azureuser@[YOUR-VM-PUBLIC-IP]
# Enter password when prompted
```

**First Connection:**
- You'll see: "The authenticity of host... can't be established"
- Type: `yes` and press Enter
- You should now be connected to your VM!

**Your prompt should change to:**
```
azureuser@vm-security-lab:~$
```

**Expected Time**: 5 minutes

### Step 9: Initial VM Security Configuration

Now that you're connected, let's harden the VM.

#### Update System Packages

```bash
# Update package lists
sudo apt update

# Upgrade all packages (this may take 5-10 minutes)
sudo apt upgrade -y

# Remove unnecessary packages
sudo apt autoremove -y
```

#### Configure Automatic Security Updates

```bash
# Install unattended-upgrades
sudo apt install unattended-upgrades -y

# Enable automatic security updates
sudo dpkg-reconfigure -plow unattended-upgrades
# Select "Yes" when prompted
```

#### Configure Firewall (UFW - Additional Layer)

Even though we have NSG, adding UFW provides defense in depth:

```bash
# Install UFW (usually pre-installed)
sudo apt install ufw -y

# Set default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (port 22)
sudo ufw allow 22/tcp

# Enable UFW
sudo ufw enable

# Check status
sudo ufw status verbose
```

Expected output:
```
Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
```

#### Install Fail2Ban (Brute Force Protection)

```bash
# Install fail2ban
sudo apt install fail2ban -y

# Start and enable fail2ban
sudo systemctl start fail2ban
sudo systemctl enable fail2ban

# Check status
sudo systemctl status fail2ban

# View protected services
sudo fail2ban-client status
```

#### Create Fail2Ban Configuration for SSH

```bash
# Create local configuration
sudo nano /etc/fail2ban/jail.local
```

Add this content:
```ini
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 3

[sshd]
enabled = true
port = ssh
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
```

Save and exit (Ctrl+X, then Y, then Enter)

```bash
# Restart fail2ban
sudo systemctl restart fail2ban

# Verify SSH jail is active
sudo fail2ban-client status sshd
```

**Expected Time**: 15 minutes

---

## Monitoring Setup

### Step 10: Create Log Analytics Workspace

Back in Azure Portal (open new browser tab, keep SSH connection open):

1. Search for **"Log Analytics workspaces"**
2. Click **"+ Create"**

3. Fill in:
   ```
   Subscription: [Your subscription]
   Resource group: rg-security-project
   Name: law-security-monitor
   Region: East US (same as resource group)
   ```

4. Click **"Review + Create"** → **"Create"**

**Via Azure CLI** (Alternative):
```bash
az monitor log-analytics workspace create \
  --resource-group rg-security-project \
  --workspace-name law-security-monitor \
  --location eastus
```

**Expected Time**: 3 minutes

### Step 11: Enable VM Insights

1. Navigate to your VM: `vm-security-lab`
2. In the left sidebar, under **"Monitoring"**, click **"Insights"**
3. Click **"Enable"** button

4. Configure:
   ```
   Data collection rule:
     Create new
     
   Data collection rule name: dcr-vm-security
   
   Log Analytics workspace: law-security-monitor
   
   Guest performance: Enabled
   Processes and dependencies: Enabled (optional)
   ```

5. Click **"Configure"**

**Deployment:**
- This installs the Azure Monitor agent on your VM
- Takes 5-10 minutes
- You'll see "Configuring monitoring..." status

**Verification:**
- Wait for "Successfully configured" message
- Refresh the Insights page
- You should start seeing performance data

**Expected Time**: 15 minutes (including deployment)

### Step 12: Configure Diagnostic Settings

1. Go to your VM
2. Click **"Diagnostic settings"** under Monitoring
3. Click **"+ Add diagnostic setting"**

4. Configure:
   ```
   Diagnostic setting name: diag-vm-security
   
   Logs:
     ☑ Administrative
     ☑ Security
     ☑ Alert
   
   Metrics:
     ☑ AllMetrics
   
   Destination details:
     ☑ Send to Log Analytics workspace
     Subscription: [Your subscription]
     Log Analytics workspace: law-security-monitor
   ```

5. Click **"Save"**

**Expected Time**: 5 minutes

---

## Alert Configuration

### Step 13: Create Action Group

Action Groups define who gets notified when an alert fires.

1. Search for **"Monitor"** in Azure Portal
2. Click **"Alerts"** in the left sidebar
3. Click **"Action groups"** at the top
4. Click **"+ Create"**

#### Basics Tab

```
Subscription: [Your subscription]
Resource group: rg-security-project
Action group name: ag-security-alerts
Display name: SecurityAlerts
```

Click **"Next: Notifications"**

#### Notifications Tab

```
Notification type: Email/SMS message/Push/Voice
Name: EmailAdmin

Configuration:
  ☑ Email
  Email address: [your-email@example.com]
  
  Optional:
  ☐ SMS
  ☐ Voice call
  ☐ Azure app Push Notifications
```

Click **"Next: Actions"** (skip this)

Click **"Next: Tags"** (skip this)

Click **"Review + create"** → **"Create"**

**Verification:**
- You'll receive a confirmation email
- Click the link in the email to verify
- Subject: "You have been added to an Azure Action Group"

**Expected Time**: 5 minutes

### Step 14: Create High CPU Alert

1. Still in **Monitor**, click **"Alerts"**
2. Click **"+ Create"** → **"Alert rule"**

#### Select Scope

1. Click **"+ Select scope"**
2. Filter by:
   ```
   Resource type: Virtual machines
   ```
3. Select: `vm-security-lab`
4. Click **"Done"**

#### Configure Condition

1. Click **"+ Add condition"**
2. Search for: `Percentage CPU`
3. Select: **"Percentage CPU"**

4. Configure:
   ```
   Threshold: Static
   Aggregation type: Average
   Operator: Greater than
   Threshold value: 80
   
   Check every: 5 minutes
   Lookback period: 5 minutes
   ```

5. Click **"Next: Actions"**

#### Configure Actions

1. Click **"+ Select action groups"**
2. Select: `ag-security-alerts`
3. Click **"Select"**

Click **"Next: Details"**

#### Configure Details

```
Subscription: [Your subscription]
Resource group: rg-security-project
Severity: 2 - Warning
Alert rule name: alert-high-cpu
Alert rule description: CPU usage exceeded 80% for 5 minutes
Enable alert rule upon creation: ☑ (checked)
Automatically resolve alerts: ☑ (checked)
```

Click **"Review + create"** → **"Create"**

**Expected Time**: 10 minutes

### Step 15: Create Failed SSH Login Alert

1. In **Monitor**, click **"Alerts"** → **"+ Create"** → **"Alert rule"**

#### Select Scope

1. Click **"+ Select scope"**
2. Filter by:
   ```
   Resource type: Log Analytics workspaces
   ```
3. Select: `law-security-monitor`
4. Click **"Done"**

#### Configure Condition

1. Click **"+ Add condition"**
2. Select: **"Custom log search"**

3. Configure query:
   ```kql
   Syslog
   | where Computer contains "vm-security-lab"
   | where Facility == "authpriv"
   | where SeverityLevel == "err"
   | where SyslogMessage contains "Failed password"
   | summarize FailedLoginCount = count() by Computer
   | where FailedLoginCount > 3
   ```

4. Alert logic:
   ```
   Based on: Number of results
   Operator: Greater than
   Threshold value: 0
   
   Frequency: 5 minutes
   ```

5. Click **"Next: Actions"**

#### Configure Actions

1. Select: `ag-security-alerts`
2. Click **"Next: Details"**

#### Configure Details

```
Severity: 1 - Error
Alert rule name: alert-failed-ssh-logins
Alert rule description: Multiple failed SSH authentication attempts detected
Enable alert rule upon creation: ☑ (checked)
```

Click **"Review + create"** → **"Create"**

**Expected Time**: 10 minutes

---

## Testing & Validation

### Step 16: Test NSG Rules

#### Test 1: Verify Current IP Works

```bash
# From your local machine
ssh azureuser@[YOUR-VM-PUBLIC-IP]
```

**Expected Result**: ✅ Connection successful

#### Test 2: Verify Different IP Fails

**Option A - Use VPN:**
1. Connect to a VPN (ProtonVPN, Windscribe, etc.)
2. Try connecting:
   ```bash
   ssh azureuser@[YOUR-VM-PUBLIC-IP]
   ```
   **Expected Result**: ❌ Connection timeout (after ~60 seconds)

**Option B - Use Azure Cloud Shell:**
1. In Azure Portal, click Cloud Shell icon (>_) at top
2. Choose Bash
3. Try connecting:
   ```bash
   ssh azureuser@[YOUR-VM-PUBLIC-IP]
   ```
   **Expected Result**: ❌ Connection timeout or refused

**Screenshot**: Capture the error message

**Expected Time**: 10 minutes

### Step 17: Test CPU Monitoring

#### Generate CPU Load

1. SSH into your VM
2. Install stress tool:
   ```bash
   sudo apt install stress -y
   ```

3. Run stress test:
   ```bash
   # Use all CPU cores for 5 minutes
   stress --cpu 2 --timeout 300s
   ```

#### Monitor in Azure

1. Go to Azure Portal → Your VM
2. Click **"Metrics"** under Monitoring
3. Select metric: **"Percentage CPU"**
4. Set time range: **Last 30 minutes**
5. Watch CPU climb to 100%

#### Wait for Alert

- Alert checks every 5 minutes
- When CPU > 80% for 5 minutes, alert fires
- You should receive email within 5-15 minutes

**Expected Email:**
```
Subject: Azure: Activated Severity: 2 alert-high-cpu

Alert: alert-high-cpu
Severity: 2 - Warning
Resource: vm-security-lab
Condition: Percentage CPU > 80.00
Value: 98.5%
```

**Screenshot**: Capture the metrics graph and email

**Expected Time**: 15 minutes

### Step 18: Test Security Logging

#### Generate Failed Login Attempts

1. From your local machine, try logging in with wrong credentials:
   ```bash
   ssh wronguser@[YOUR-VM-PUBLIC-IP]
   # Type random password 3-4 times
   ```

2. Try with correct username but wrong password (if using password auth):
   ```bash
   ssh azureuser@[YOUR-VM-PUBLIC-IP]
   # Type wrong password 3-4 times
   ```

#### Query Logs

1. Go to Azure Portal
2. Navigate to: **Log Analytics workspaces** → `law-security-monitor`
3. Click **"Logs"** in left sidebar

4. Run this query:
   ```kql
   Syslog
   | where Computer contains "vm-security-lab"
   | where Facility == "authpriv"
   | where SeverityLevel == "err"
   | where SyslogMessage contains "Failed password"
   | project TimeGenerated, Computer, ProcessName, SyslogMessage
   | order by TimeGenerated desc
   | take 20
   ```

5. Click **"Run"**

**Expected Results**: You should see your failed login attempts

**Screenshot**: Capture the query results

**Expected Time**: 10 minutes

### Step 19: Verify Fail2Ban

Check if fail2ban caught the failed attempts:

```bash
# SSH into your VM
ssh azureuser@[YOUR-VM-PUBLIC-IP]

# Check banned IPs
sudo fail2ban-client status sshd

# View fail2ban logs
sudo tail -f /var/log/fail2ban.log
```

**Expected Output**:
```
Status for the jail: sshd
|- Filter
|  |- Currently failed: 0
|  |- Total failed:     5
|  `- File list:        /var/log/auth.log
`- Actions
   |- Currently banned: 1
   |- Total banned:     1
   `- Banned IP list:   [YOUR-TEST-IP]
```

**Expected Time**: 5 minutes

---

## Troubleshooting

### Common Issues and Solutions

#### Issue 1: Cannot Connect to VM

**Symptoms**: SSH connection times out or is refused

**Solutions**:

1. **Check VM Status**:
   - Go to VM in portal
   - Ensure status is "Running"
   - If stopped, click "Start"

2. **Verify NSG Rules**:
   - Go to VM → Networking
   - Check if SSH rule exists
   - Verify your current IP hasn't changed (Google "what's my IP")
   - Update NSG rule with new IP if needed

3. **Check Public IP**:
   - VM → Overview
   - Verify Public IP address exists
   - Try pinging: `ping [VM-PUBLIC-IP]`

4. **Test Port Connectivity**:
   ```bash
   # Windows PowerShell
   Test-NetConnection -ComputerName [VM-IP] -Port 22
   
   # Mac/Linux
   nc -zv [VM-IP] 22
   ```

#### Issue 2: No Alert Emails Received

**Solutions**:

1. **Check Spam Folder**

2. **Verify Action Group**:
   - Monitor → Action groups → `ag-security-alerts`
   - Check email is confirmed (green checkmark)
   - Send test notification

3. **Check Alert Rule Status**:
   - Monitor → Alerts → Alert rules
   - Verify alert is "Enabled"
   - Check "Alert condition" shows the criteria

4. **Verify Condition Met**:
   - Alerts → Alert rules → Click your alert
   - View "History" tab
   - Check if alert actually fired

#### Issue 3: No Data in Log Analytics

**Solutions**:

1. **Wait 10-15 minutes** for initial data collection

2. **Verify Agent Installation**:
   - VM → Extensions + applications
   - Should see "AzureMonitorLinuxAgent"
   - Status should be "Provisioning succeeded"

3. **Check Diagnostic Settings**:
   - VM → Diagnostic settings
   - Verify logs are enabled
   - Confirm Log Analytics workspace is selected

4. **Test with Simple Query**:
   ```kql
   Heartbeat
   | where Computer contains "vm-security-lab"
   | take 10
   ```

#### Issue 4: High Costs

**Solutions**:

1. **Enable Auto-Shutdown**:
   - VM → Auto-shutdown
   - Set daily shutdown time

2. **Stop VM When Not Using**:
   - VM → Overview → Click "Stop"
   - Deallocates resources (no compute charges)

3. **Monitor Costs**:
   - Search "Cost Management"
   - Set up budget alerts

4. **Right-Size VM**:
   - Use B1s size (cheapest)
   - Only scale up when needed

#### Issue 5: SSH Key Permission Denied

**Solutions**:

```bash
# Linux/Mac - Fix key permissions
chmod 400 ~/path/to/your-key.pem

# Windows PowerShell - Fix key permissions
icacls .\your-key.pem /inheritance:r
icacls .\your-key.pem /grant:r "$env:USERNAME:(R)"

# If key is lost, reset to password
# Azure Portal → VM → Reset password
```

---

## Cost Management

### Estimated Monthly Costs

```
VM (B1s): $10.95/month
- Stopped/deallocated: $0
- Running 24/7: $10.95
- Running 8 hours/day: $3.65

Log Analytics: $0-5/month
- First 5GB free
- Typically use < 1GB for this project

Storage (OS Disk): $1.54/month
- Standard SSD 30GB

Public IP (Static): $3.65/month
- Dynamic: $0 (but changes when VM stops)

Total if running 24/7: ~$16/month
Total with auto-shutdown: ~$5/month
Free tier credits: $200 (covers 12+ months)
```

### Cost Optimization Tips

1. **Use Auto-Shutdown**:
   ```
   VM → Auto-shutdown → Enable
   Set to shutdown at 7 PM daily
   ```

2. **Stop VM When Not In Use**:
   ```bash
   # Stop VM (deallocates compute resources)
   az vm deallocate \
     --resource-group rg-security-project \
     --name vm-security-lab
   
   # Start VM when needed
   az vm start \
     --resource-group rg-security-project \
     --name vm-security-lab
   ```

3. **Use Dynamic Public IP**:
   - Costs $0 when VM is stopped
   - Changes each time VM starts (acceptable for lab)
   - Only use Static IP if you need consistent address

4. **Set Budget Alerts**:
   ```
   Search "Budgets" in Azure Portal
   Create budget: $20/month
   Alert at 80% ($16)
   Alert at 100% ($20)
   ```

5. **Delete When Done**:
   ```bash
   # Delete entire resource group (all resources)
   az group delete \
     --name rg-security-project \
     --yes \
     --no-wait
   ```

### Monitor Your Spending

1. **Cost Analysis**:
   - Search "Cost Management"
   - View by: Resource group
   - Filter: rg-security-project
   - Check daily/weekly costs

2. **Set Up Cost Alerts**:
   - Cost Management → Budgets
   - Create budget alert
   - Get email when approaching limit

3. **Review Free Credits**:
   - Azure Portal → Subscriptions
   - Check remaining free credits
   - Monitor expiration date

---

## Post-Setup Verification Checklist

Use this checklist to ensure everything is configured correctly:

### Infrastructure
- [ ] Resource Group created: `rg-security-project`
- [ ] Virtual Network created: `vnet-security-lab` (10.0.0.0/16)
- [ ] Subnet created: `subnet-default` (10.0.1.0/24)
- [ ] NSG created: `nsg-vm-security`
- [ ] VM created: `vm-security-lab` (Running status)
- [ ] Public IP assigned to VM
- [ ] All resources in same region

### Security
- [ ] NSG rule: Allow SSH from my IP (Priority 100)
- [ ] NSG rule: Allow HTTPS (Priority 110)
- [ ] NSG rule: Default deny (Priority 65000)
- [ ] SSH key or strong password configured
- [ ] VM system packages updated
- [ ] UFW firewall enabled on VM
- [ ] Fail2Ban installed and running
- [ ] Auto-shutdown configured

### Monitoring
- [ ] Log Analytics workspace created: `law-security-monitor`
- [ ] VM Insights enabled
- [ ] Azure Monitor agent installed on VM
- [ ] Diagnostic settings configured
- [ ] Data flowing to Log Analytics (wait 15 min)

### Alerts
- [ ] Action Group created: `ag-security-alerts`
- [ ] Email notification configured and verified
- [ ] High CPU alert created and enabled
- [ ] Failed SSH login alert created and enabled
- [ ] Test alerts fired successfully

### Testing
- [ ] Successfully connected from authorized IP
- [ ] Connection blocked from unauthorized IP
- [ ] CPU stress test triggered alert
- [ ] Alert email received
- [ ] Failed login attempts logged
- [ ] Log Analytics queries returning data
- [ ] Fail2Ban catching failed attempts

### Documentation
- [ ] Screenshots taken of all configurations
- [ ] Architecture diagram created
- [ ] Public IP address documented
- [ ] Username and authentication method noted
- [ ] Alert configurations documented

---

## Advanced Configurations (Optional)

### Enable Just-In-Time (JIT) VM Access

JIT provides temporary access to VMs, reducing attack surface.

1. Search for **"Microsoft Defender for Cloud"**
2. Go to **"Workload protections"**
3. Click **"Just-in-time VM access"**
4. Select your VM
5. Click **"Enable JIT"**
6. Configure:
   ```
   Port: 22
   Protocol: TCP
   Max request time: 3 hours
   Allowed source IPs: My IP
   ```

**Note**: This requires Microsoft Defender for Cloud (additional cost)

### Configure Network Watcher

Network Watcher provides network diagnostic and monitoring tools.

1. Search for **"Network Watcher"**
2. If not enabled, click **"Enable Network Watcher"**
3. Select your region

#### NSG Flow Logs

Track all traffic through your NSG:

1. Network Watcher → **"NSG flow logs"**
2. Click **"+ Create"**
3. Select your NSG: `nsg-vm-security`
4. Configure storage account (creates new one)
5. Enable Traffic Analytics (optional)
6. Click **"Create"**

**Benefits**:
- See all connection attempts
- Identify attack patterns
- Audit network access

### Install Additional Security Tools

#### Install and Configure Auditd

```bash
# SSH into your VM
ssh azureuser@[YOUR-VM-PUBLIC-IP]

# Install auditd
sudo apt install auditd audispd-plugins -y

# Start auditd
sudo systemctl enable auditd
sudo systemctl start auditd

# Add rules to monitor SSH access
sudo auditctl -w /var/log/auth.log -p wa -k auth_log
sudo auditctl -w /etc/ssh/sshd_config -p wa -k sshd_config

# Make rules persistent
sudo sh -c "auditctl -l >> /etc/audit/rules.d/audit.rules"

# View audit logs
sudo ausearch -k auth_log
```

#### Install Lynis Security Auditing Tool

```bash
# Install Lynis
sudo apt install lynis -y

# Run security audit
sudo lynis audit system

# View report
sudo cat /var/log/lynis-report.dat
```

This will give you security recommendations for hardening your VM.

#### Install and Configure AIDE (Advanced Intrusion Detection)

```bash
# Install AIDE
sudo apt install aide aide-common -y

# Initialize AIDE database (takes 5-10 minutes)
sudo aideinit

# Copy database
sudo cp /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Run check
sudo aide --check

# Set up daily checks
echo "0 5 * * * root /usr/bin/aide --check" | sudo tee -a /etc/crontab
```

### Enable Azure Backup

Protect your VM with automated backups:

1. Go to your VM
2. Click **"Backup"** under Operations
3. Click **"Enable backup"**
4. Configure:
   ```
   Recovery Services vault: Create new
   Vault name: rsv-security-backup
   Backup policy: DefaultPolicy (daily at 10 PM)
   ```
5. Click **"Enable Backup"**

**Backup policy options**:
- Daily backup at specified time
- Retention: 30 days (configurable)
- Instant restore snapshots

### Configure Update Management

Automate patch management:

1. Go to your VM
2. Click **"Update Management"** under Operations
3. Click **"Enable"**
4. Select Log Analytics workspace
5. Configure schedule:
   ```
   Name: Weekly-Security-Updates
   Schedule: Weekly on Sunday at 2 AM
   Updates: Critical and Security only
   Pre/Post scripts: None
   Maintenance window: 2 hours
   Reboot: If required
   ```

---

## Automation Scripts

### Bash Script: Quick Health Check

Create a script to check VM health:

```bash
#!/bin/bash
# save as: vm-health-check.sh

echo "=== Azure VM Security Health Check ==="
echo "Date: $(date)"
echo ""

echo "1. System Uptime:"
uptime
echo ""

echo "2. CPU Usage:"
top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print 100 - $1"%"}'
echo ""

echo "3. Memory Usage:"
free -h
echo ""

echo "4. Disk Usage:"
df -h /
echo ""

echo "5. Failed Login Attempts (last 10):"
sudo grep "Failed password" /var/log/auth.log | tail -10
echo ""

echo "6. UFW Firewall Status:"
sudo ufw status
echo ""

echo "7. Fail2Ban Status:"
sudo fail2ban-client status
echo ""

echo "8. Active SSH Connections:"
who
echo ""

echo "9. System Updates Available:"
sudo apt update > /dev/null 2>&1
apt list --upgradable 2>/dev/null | wc -l
echo ""

echo "=== Health Check Complete ==="
```

Make it executable and run:
```bash
chmod +x vm-health-check.sh
./vm-health-check.sh
```

### PowerShell Script: Manage VM from Local Machine

```powershell
# save as: manage-vm.ps1

# Variables
$resourceGroup = "rg-security-project"
$vmName = "vm-security-lab"

# Menu
function Show-Menu {
    Clear-Host
    Write-Host "=== Azure VM Management ==="
    Write-Host "1. Check VM Status"
    Write-Host "2. Start VM"
    Write-Host "3. Stop VM"
    Write-Host "4. Restart VM"
    Write-Host "5. Get VM Public IP"
    Write-Host "6. View Recent Alerts"
    Write-Host "7. Check VM Costs"
    Write-Host "Q. Quit"
}

# Functions
function Get-VMStatus {
    az vm get-instance-view `
        --resource-group $resourceGroup `
        --name $vmName `
        --query "instanceView.statuses[?starts_with(code, 'PowerState/')].displayStatus" `
        --output tsv
}

function Start-AzVM {
    Write-Host "Starting VM..."
    az vm start --resource-group $resourceGroup --name $vmName
    Write-Host "VM started successfully!"
}

function Stop-AzVM {
    Write-Host "Stopping VM..."
    az vm deallocate --resource-group $resourceGroup --name $vmName
    Write-Host "VM stopped and deallocated!"
}

function Restart-AzVM {
    Write-Host "Restarting VM..."
    az vm restart --resource-group $resourceGroup --name $vmName
    Write-Host "VM restarted successfully!"
}

function Get-PublicIP {
    az vm show `
        --resource-group $resourceGroup `
        --name $vmName `
        --show-details `
        --query "publicIps" `
        --output tsv
}

# Main loop
do {
    Show-Menu
    $choice = Read-Host "Enter your choice"
    
    switch ($choice) {
        '1' { 
            Write-Host "VM Status: $(Get-VMStatus)"
            pause
        }
        '2' { 
            Start-AzVM
            pause
        }
        '3' { 
            Stop-AzVM
            pause
        }
        '4' { 
            Restart-AzVM
            pause
        }
        '5' { 
            Write-Host "Public IP: $(Get-PublicIP)"
            pause
        }
        '6' { 
            az monitor metrics alert list `
                --resource-group $resourceGroup `
                --output table
            pause
        }
        '7' {
            az consumption usage list `
                --output table
            pause
        }
    }
} until ($choice -eq 'Q')
```

Run with:
```powershell
.\manage-vm.ps1
```

### Azure CLI: Complete Deployment Script

Deploy everything with one script:

```bash
#!/bin/bash
# save as: deploy-security-lab.sh

# Variables
RESOURCE_GROUP="rg-security-project"
LOCATION="eastus"
VNET_NAME="vnet-security-lab"
SUBNET_NAME="subnet-default"
NSG_NAME="nsg-vm-security"
VM_NAME="vm-security-lab"
VM_SIZE="Standard_B1s"
VM_IMAGE="Ubuntu2204"
ADMIN_USERNAME="azureuser"
MY_IP=$(curl -s https://api.ipify.org)

echo "=== Azure Security Lab Deployment ==="
echo "Location: $LOCATION"
echo "Your IP: $MY_IP"
echo ""

# Create Resource Group
echo "Creating Resource Group..."
az group create \
    --name $RESOURCE_GROUP \
    --location $LOCATION

# Create Virtual Network
echo "Creating Virtual Network..."
az network vnet create \
    --resource-group $RESOURCE_GROUP \
    --name $VNET_NAME \
    --address-prefix 10.0.0.0/16 \
    --subnet-name $SUBNET_NAME \
    --subnet-prefix 10.0.1.0/24

# Create NSG
echo "Creating Network Security Group..."
az network nsg create \
    --resource-group $RESOURCE_GROUP \
    --name $NSG_NAME

# Add NSG Rules
echo "Configuring NSG Rules..."
az network nsg rule create \
    --resource-group $RESOURCE_GROUP \
    --nsg-name $NSG_NAME \
    --name Allow-SSH-MyIP \
    --priority 100 \
    --source-address-prefixes $MY_IP/32 \
    --destination-port-ranges 22 \
    --protocol Tcp \
    --access Allow

az network nsg rule create \
    --resource-group $RESOURCE_GROUP \
    --nsg-name $NSG_NAME \
    --name Allow-HTTPS \
    --priority 110 \
    --source-address-prefixes '*' \
    --destination-port-ranges 443 \
    --protocol Tcp \
    --access Allow

# Create VM
echo "Creating Virtual Machine..."
az vm create \
    --resource-group $RESOURCE_GROUP \
    --name $VM_NAME \
    --location $LOCATION \
    --image $VM_IMAGE \
    --size $VM_SIZE \
    --admin-username $ADMIN_USERNAME \
    --generate-ssh-keys \
    --vnet-name $VNET_NAME \
    --subnet $SUBNET_NAME \
    --nsg $NSG_NAME \
    --public-ip-sku Standard

# Enable Auto-Shutdown
echo "Configuring Auto-Shutdown..."
az vm auto-shutdown \
    --resource-group $RESOURCE_GROUP \
    --name $VM_NAME \
    --time 1900

# Get VM Public IP
PUBLIC_IP=$(az vm show \
    --resource-group $RESOURCE_GROUP \
    --name $VM_NAME \
    --show-details \
    --query publicIps \
    --output tsv)

echo ""
echo "=== Deployment Complete ==="
echo "VM Public IP: $PUBLIC_IP"
echo "Username: $ADMIN_USERNAME"
echo "Connect with: ssh $ADMIN_USERNAME@$PUBLIC_IP"
echo ""
echo "Next steps:"
echo "1. Create Log Analytics workspace"
echo "2. Enable VM Insights"
echo "3. Configure alerts"
```

Make executable and run:
```bash
chmod +x deploy-security-lab.sh
./deploy-security-lab.sh
```

---

## Documentation Templates

### Daily Operations Log Template

Keep track of your activities:

```markdown
# Azure Security Lab - Operations Log

## Date: [YYYY-MM-DD]

### Activities Performed
- [ ] Checked VM status
- [ ] Reviewed security alerts
- [ ] Updated system packages
- [ ] Checked failed login attempts
- [ ] Reviewed NSG flow logs

### Security Events
| Time | Event Type | Source IP | Action Taken |
|------|-----------|-----------|--------------|
| 14:30 | Failed SSH | 192.0.2.45 | Blocked by NSG |
| 15:15 | High CPU | N/A | Investigated, normal activity |

### Changes Made
- Updated NSG rule to allow IP: 203.0.113.10
- Increased alert threshold to 85%
- Added new monitoring query

### Issues Encountered
- None

### Notes
- VM running smoothly
- No security incidents
- Cost tracking: $2.15 today
```

### Incident Response Template

```markdown
# Security Incident Report

## Incident ID: INC-001
**Date**: [YYYY-MM-DD]
**Time**: [HH:MM UTC]
**Severity**: [Low/Medium/High/Critical]
**Status**: [Open/In Progress/Resolved]

### Summary
Brief description of the incident.

### Timeline
- **14:00** - Alert received: High CPU usage
- **14:05** - Investigation started
- **14:15** - Root cause identified: Stress test
- **14:20** - Incident resolved

### Root Cause
Describe what caused the incident.

### Impact
- Services affected: None
- Users affected: 0
- Duration: 20 minutes

### Actions Taken
1. Connected to VM via SSH
2. Checked running processes
3. Identified stress test process
4. Terminated process
5. Verified normal operation

### Prevention Measures
- Document stress test procedures
- Create separate test environment
- Adjust alert thresholds

### Lessons Learned
- Need better communication about testing
- Alert worked as expected
- Response time: 5 minutes (good)
```

---

## Next Steps & Career Path

### Immediate Next Projects

1. **Expand to Multi-VM Environment**
   - Add load balancer
   - Configure multiple VMs
   - Implement high availability

2. **Implement Infrastructure as Code**
   - Learn Terraform or Bicep
   - Version control your infrastructure
   - Automate deployments

3. **Add Application Layer**
   - Deploy web application
   - Configure Application Gateway
   - Implement WAF (Web Application Firewall)

4. **Enhance Monitoring**
   - Set up Azure Sentinel (SIEM)
   - Create custom dashboards
   - Implement threat detection

### Recommended Learning Path

1. **Azure Certifications**:
   - AZ-900: Azure Fundamentals
   - AZ-104: Azure Administrator
   - AZ-500: Azure Security Engineer
   - SC-200: Security Operations Analyst

2. **Security Certifications**:
   - CompTIA Security+
   - Certified Ethical Hacker (CEH)
   - CISSP (with experience)

3. **DevSecOps Skills**:
   - Learn Git/GitHub
   - CI/CD pipelines (Azure DevOps)
   - Container security (Docker, Kubernetes)
   - Infrastructure as Code (Terraform)

### Resources

**Official Microsoft Learn Paths**:
- Azure Security: https://learn.microsoft.com/training/azure/
- Security best practices
- Hands-on labs (free)

**Community Resources**:
- Azure Community Discord
- Reddit: r/AZURE, r/cybersecurity
- Azure Security Podcast
- Cloud Security Alliance

**Books**:
- "Azure Security Best Practices" by Microsoft
- "The Phoenix Project" (DevOps culture)
- "Hands-On Azure Security"

---

## Conclusion

Congratulations! You've successfully:

✅ Built a secure Azure environment from scratch
✅ Implemented network security with NSGs
✅ Configured comprehensive monitoring and alerting
✅ Hardened a Linux VM with multiple security layers
✅ Tested and validated all security controls
✅ Documented your work professionally

### What Makes This Project Stand Out

- **Production-grade security**: Not just a demo, but real security practices
- **Defense in depth**: Multiple security layers (NSG, UFW, Fail2Ban)
- **Monitoring & observability**: Proactive threat detection
- **Documentation**: Professional documentation shows attention to detail
- **Testing**: Validated that everything actually works

### Using This in Interviews

**When asked about security experience**:
> "I designed and implemented a secure Azure environment with multiple security layers. I configured network security groups following the principle of least privilege, implemented monitoring with Azure Monitor and Log Analytics, and set up automated alerting for security events. I validated the security controls by testing unauthorized access attempts and confirmed all alerts triggered correctly."

**When asked about troubleshooting**:
> "In my Azure security project, I encountered connection issues that I systematically debugged. I verified NSG rules, checked VM status, validated network connectivity, and ultimately traced it to my dynamic IP changing. This taught me the importance of understanding the full network stack and proper diagnostic methodology."

**When asked about monitoring**:
> "I implemented comprehensive monitoring using Azure Monitor and Log Analytics. I wrote KQL queries to detect failed authentication attempts and set up automated alerts with proper severity levels. When alerts fired, they provided actionable information that allowed quick response and investigation."

---

## Support & Feedback

### Getting Help

**Azure Support**:
- Azure Portal: Click "?" icon → "Help + support"
- Community forums: Microsoft Q&A
- Stack Overflow: Tag questions with `azure`

**This Project**:
- GitHub Issues: [Your repo issues page]
- Email: [Your email]
- LinkedIn: [Your LinkedIn]

### Contributing

Found an improvement? Suggestions:
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

### Feedback

Your feedback helps improve this guide! Please share:
- What worked well
- What was confusing
- What's missing
- Suggestions for improvement

---

## Appendix

### A. Useful Azure CLI Commands

```bash
# List all resources in resource group
az resource list --resource-group rg-security-project --output table

# Get VM details
az vm show --resource-group rg-security-project --name vm-security-lab

# List all NSG rules
az network nsg rule list --resource-group rg-security-project --nsg-name nsg-vm-security --output table

# Check VM metrics
az monitor metrics list --resource [VM-RESOURCE-ID] --metric "Percentage CPU"

# Export NSG rules
az network nsg rule list --resource-group rg-security-project --nsg-name nsg-vm-security > nsg-rules.json
```

### B. Useful KQL Queries

```kql
// Top CPU consuming processes
Perf
| where Computer contains "vm-security-lab"
| where CounterName == "% Processor Time"
| where InstanceName != "_Total"
| summarize avg(CounterValue) by InstanceName
| order by avg_CounterValue desc
| take 10

// Network connections
VMConnection
| where Computer contains "vm-security-lab"
| summarize Connections = count() by RemoteIp
| order by Connections desc

// Disk performance
Perf
| where Computer contains "vm-security-lab"
| where CounterName == "Disk Transfers/sec"
| summarize avg(CounterValue) by bin(TimeGenerated, 5m)
| render timechart
```

### C. Keyboard Shortcuts

**Azure Portal**:
- `G + N`: Go to Notifications
- `G + /`: Go to Search
- `/`: Focus search box
- `Ctrl + /`: Open keyboard shortcuts help

**SSH Shortcuts**:
- `Ctrl + D`: Logout
- `Ctrl + L`: Clear screen
- `Ctrl + R`: Search command history
- `Ctrl + C`: Cancel current command

### D. Glossary

| Term | Definition |
|------|------------|
| **NSG** | Network Security Group - Azure's virtual firewall |
| **VNet** | Virtual Network - Isolated network in Azure |
| **KQL** | Kusto Query Language - Query language for Azure logs |
| **SSH** | Secure Shell - Encrypted remote access protocol |
| **UFW** | Uncomplicated Firewall - Linux firewall tool |
| **CIDR** | Classless Inter-Domain Routing - IP addressing notation |
| **JIT** | Just-In-Time - Temporary VM access feature |
| **SIEM** | Security Information and Event Management |
| **IaaS** | Infrastructure as a Service |

---

**Document Version**: 1.0  
**Last Updated**: October 2025  
**Author**: Endurance Joseph Ogbeide  
**License**: MIT

---

*This guide is part of the Azure Security Setup Project. For the latest version, visit: [Your GitHub Repo URL]*
