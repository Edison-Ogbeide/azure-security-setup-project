# Azure Basic Security Setup (Entry Project)

## Summary
Small Azure demo: created a single VM, configured Network Security Group (NSG) rules to restrict management access, enabled Azure Monitor/agent, and created a basic alert rule. Includes an audit script that collects resource and security configuration status.

## What I did
- Created Resource Group, Virtual Machine (Ubuntu), and Network Security Group.
- Restricted SSH to a single IP address.
- Enabled Azure Monitor (VM Insights) and installed the Azure Monitor Agent.
- Created a simple metric alert (CPU or Heartbeat) with an action group.

## Files
- `audit.sh` — Azure CLI script to collect configuration status (generates `results.md`).
- `results.md` — Example output from a local run.
- `screenshots/` — Visual evidence (no credentials included).

## How to run the audit (requires az cli)
```bash
chmod +x audit.sh
./audit.sh
