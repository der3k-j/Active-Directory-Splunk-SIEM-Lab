# Active Directory & Splunk SIEM Detection Lab

## Project Overview
This project demonstrates the design, deployment, and configuration of an isolated enterprise Active Directory environment integrated with Splunk Enterprise SIEM. The lab simulates real-world Security Operations Center (SOC) telemetry pipelines, advanced GPO audit logging policies, brute-force attack simulations, and custom Search Processing Language (SPL) detection engineering.

---

## Lab Architecture & Network Topology

- **Network Subnet:** `192.168.10.0/24` (VirtualBox Host-Only Isolated Network, Gateway: `192.168.10.1`, No DHCP)
- **Domain Controller (`WinDC`):** Windows Server 2022 | IP: `192.168.10.10` | Domain: `lab.local`
  - Active Directory Domain Services (AD DS), Domain DNS, Group Policy Management, Splunk Enterprise (Receiver Port 9997)
- **Workstation (`Win10`):** Windows 10 Workstation | IP: `192.168.10.20` | Domain-joined (`lab.local`)
  - Generates Windows Event Logs, Splunk Universal Forwarder Agent

---

## Phase 1: Network Isolation & Identity Provisioning

### 1. Host-Only Network Setup
Configured an isolated VirtualBox Host-Only network adapter (`192.168.10.1/24`) with DHCP disabled to enforce static IP addressing and prevent conflict with production networks.

### 2. VM 1 (WinDC) Static IP Verification
Configured Windows Server 2022 with static IP `192.168.10.10` and self-referencing primary DNS (`192.168.10.10`).

![WinDC IP Config](docs/screenshots/01_ipconfig_windc.png)

### 3. VM 2 (Win10) Static IP & Domain Join Verification
Configured Windows 10 with static IP `192.168.10.20` pointing to `192.168.10.10` for DNS, then joined the domain `lab.local`.

![Win10 IP Config](docs/screenshots/02_ipconfig_win10.png)

![Domain Join Success](docs/screenshots/03_domain_join_success.png)

---

## Phase 2: Enhanced Telemetry via GPO Audit Policies

Configured the **Default Domain Policy** via Group Policy Management (`gpmc.msc`) on `WinDC` to enforce audit logging baselines across domain endpoints:

- **Audit Logon (Success & Failure):** Records all login attempts (Event IDs 4624 & 4625).
- **Audit Privilege Use (Success & Failure):** Tracks elevated rights and administrative actions.
- **Audit File System (Success & Failure):** Logs unauthorized sensitive file modifications/access.

Policy update enforced across endpoints using `gpupdate /force`.

![GPO Audit Policy Configuration](docs/screenshots/04_gpo_audit_config.png)

---

## Phase 3: Ingestion Pipeline & Splunk Universal Forwarder

### 1. Splunk Enterprise Receiving Setup
Installed Splunk Enterprise on `WinDC` and opened TCP receiving port **9997** (`Settings -> Forwarding and receiving`).

![Splunk Receiver Listening Port 9997](docs/screenshots/05_splunk_listening_port.png)

### 2. Universal Forwarder Log Forwarding (`inputs.conf`)
Installed Splunk Universal Forwarder on `Win10` pointing to indexer `192.168.10.10:9997`. Configured `inputs.conf` to stream local Windows Security logs:

```ini
[WinEventLog://Security]
disabled = 0
index = main

--- 
