# SOC-Home-Lab
This repository documents the design, deployment, and troubleshooting of a Security Operations Center (SOC) home lab built using virtualized infrastructure. The lab focuses on realistic network segmentation, attack simulation, and security monitoring.

# 🛡️ SOC Home Lab – pfSense + Security Onion

This repository documents the design, deployment, and troubleshooting of a Security Operations Center (SOC) home lab built using virtualized infrastructure. The lab focuses on realistic network segmentation, attack simulation, and security monitoring.

---

## 🎯 Objectives

- Simulate enterprise-style network segmentation
- Deploy a stateful firewall using pfSense
- Monitor east-west and north-south traffic using Security Onion
- Generate and analyze attack traffic
- Practice SOC alert triage and investigation
- Demonstrate troubleshooting and problem-solving skills

---

## 🧱 Lab Architecture

### Network Segments

| Network | Subnet | Purpose |
|------|------|------|
| DMZ | 192.168.10.0/24 | Attacker network (Kali / Parrot) |
| Internal | 192.168.20.0/24 | Victim machines (Windows client) |
| SecOps | 192.168.30.0/24 | Security Onion management |
| Monitor/TAP | No IP | Passive traffic monitoring |

### Key Components

- **pfSense** – Firewall and router
- **Security Onion** – SIEM / IDS / NSM
- **Kali Linux** – Attack simulation
- **Windows Client** – Victim endpoint
- **VMware Workstation** – Virtualization platform

> 📸 *Insert network diagram screenshot here*

---

## 🖥️ Phase 1 – Virtual Machine Creation

### VMware Network Configuration

- **WAN**: NAT (VMnet8)
- **DMZ**: LAN Segment – `DMZ`
- **Internal**: LAN Segment – `INTERNAL`
- **SecOps**: LAN Segment – `SECOPS`
- **Monitor**: LAN Segment – `TAP`

Each LAN Segment is isolated and not connected to the host system.

> 📸               Internet
                       |
                   [ WAN / NAT ]
                       |
                   ┌── pfSense ──┐(Used as DHCP)
                   │             │
          ┌────────┴──────┬──────┴────────┐
          │                │                │
     [ DMZ ]          [ Internal ]      [ SecOps ]   
   (Attacker)          (Victim)      (SIEM / EDR)
  192.168.10.0/24   192.168.20.0/24  192.168.30.0/24
  192.168.10.1       192.168.20.1     192.168.30.1
                                             |
                                         [Monitor]

---

## 🔥 Phase 2 – pfSense Installation & Interface Setup

### pfSense Interfaces

| Interface | IP Address |
|--------|------------|
| WAN | DHCP |
| DMZ | 192.168.10.1/24 |
| INTERNAL | 192.168.20.1/24 |
| SECOPS | 192.168.30.1/24 |

- All interfaces configured with **static IPv4**
- DHCP enabled on DMZ, INTERNAL, and SECOPS
- IPv6 disabled
- Bogon and private network blocking disabled on internal interfaces

> 📸

---

## 🔐 Phase 3 – Firewall Rules

### DMZ Rules
Allow DMZ → Internal (Any Protocol)
Block DMZ → SecOps

Purpose:
- Enable attack simulation
- Prevent attackers from accessing SOC tools

### Internal Rules
Allow Internal → Internet
Allow Internal → SecOps (HTTPS)

### SecOps Rules
Allow SecOps → Anywhere (Admin Access)
Allow SecOps → pfSense (HTTPS)

> 📸 

---

## 👁️ Phase 4 – Security Onion Installation

Security Onion was installed as a **standalone VM** (not on top of another OS).

### NIC Configuration

| NIC | Network |
|---|---|
| Management | SECOPS |
| Monitoring | INTERNAL |

Important:
- Monitoring NIC has **no IP address**
- No firewall rules are applied to the TAP interface

> 📸 

---

## 📊 Phase 5 – Security Onion Configuration

- Accessed via web UI (`https://<secops-ip>`)
- Verified services using `so-status`
- Enabled:
  - Zeek
  - Suricata
  - PCAP capture
- Alerts generated through passive monitoring (agentless)

> 📸

---

## ⚔️ Phase 6 – Attack Simulation

Attacks launched from DMZ attacker machine:
- Network scanning (Nmap)
- Credential attacks
- Service enumeration

Traffic observed in Security Onion:
- Zeek connection logs
- Suricata alerts
- PCAP analysis

> 📸

---

## 🛠️ Phase 7 – Troubleshooting & Lessons Learned

### Key Issues Encountered

#### 1. Firewall Rule Matching Failures
- DMZ → INTERNAL rules failed due to subnet object mismatches
- Verified by testing DMZ → ANY
- Resolved by validating interface subnet definitions and DHCP assignments

#### 2. Interface Directionality Confusion
- Learned pfSense evaluates rules on **ingress interfaces**
- Corrected rule placement accordingly

#### 3. Windows Host Firewall Blocking ICMP
- Temporarily disabled Windows firewall to validate network path
- Later re-enabled with scoped rules

#### 4. Over-Tightening Early
- Temporarily relaxed rules to ensure visibility
- Planned later hardening phase

> This troubleshooting process mirrors real SOC and network engineering workflows: isolate the layer, validate assumptions, test incrementally, and document outcomes.

---

## 📌 Future Improvements

- Add Elastic Agent for endpoint telemetry
- Introduce Active Directory domain
- Implement IDS-only vs IPS comparison
- Add Sigma rule testing
- Gradually harden firewall rules

---

## 🧠 Skills Demonstrated

- Network segmentation
- Stateful firewall configuration
- IDS/NSM deployment
- Attack simulation
- Traffic analysis
- SOC-style troubleshooting
- Documentation and communication

---

## 📎 Disclaimer

This lab is for educational and defensive security purposes only.

