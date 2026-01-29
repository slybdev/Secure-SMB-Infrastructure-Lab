

# 🛡️ Secure SMB Infrastructure Lab

### End-to-End Cybersecurity, Network Segmentation & Monitoring Project

## Project Overview

This project demonstrates the design, implementation, and documentation of a secure Small-to-Medium Business (SMB) IT infrastructure using enterprise security principles.

The lab simulates a real-world business environment with multiple departments, segmented networks, centralized identity management, firewall enforcement, centralized logging, remote access VPN, and a DMZ for public-facing services.

The goal of this project is to showcase practical, hands-on experience in:

* Network security architecture
* Firewall policy design
* Active Directory and identity management
* Centralized logging and monitoring (SOC-lite)
* Secure remote access (VPN)
* Defense-in-depth and lateral movement prevention

This environment was built to reflect how real SMBs design secure internal networks while supporting business operations and remote users.

---

## High-Level Architecture Diagram

> *Full enterprise-style network segmentation and security architecture*


*(LAN, Server VLAN, Marketing VLAN, DMZ, VPN, pfSense Firewall, Splunk, Internet)*
<img width="1450" height="692" alt="Screenshot 2026-01-29 194347" src="https://github.com/user-attachments/assets/6546ada0-4f81-43e1-809e-2d0de15f5bc6" />

---

## Network Architecture Summary

The environment is designed using a layered, segmented architecture to reduce attack surface and prevent lateral movement.

### VLAN & Subnet Design

| Zone / VLAN    | Subnet          | Purpose                                              |
| -------------- | --------------- | ---------------------------------------------------- |
| LAN (IT Team)  | 192.168.10.0/24 | IT staff workstations with administrative access     |
| Server VLAN    | 192.168.20.0/24 | Domain Controller, Splunk, business-critical servers |
| Marketing VLAN | 192.168.30.0/24 | End-user workstations with restricted access         |
| DMZ            | 192.168.40.0/24 | Public-facing web servers                            |
| VPN Subnet     | 192.168.50.0/24 | Remote Marketing users                               |

All VLANs are routed and controlled through a pfSense firewall, enforcing strict security boundaries between departments and server infrastructure.

---

## Core Components

### Firewall & Network Security (pfSense)

* Central firewall routing all VLANs
* Inter-VLAN firewall rules enforcing least privilege
* DMZ isolation for public-facing services
* NAT and controlled Internet access
* VPN for secure remote user access
* Firewall logging forwarded to Splunk

---

### Identity & Access Management (Active Directory)

* Windows Server 2022 Domain Controller
* Centralized authentication and DNS
* Organizational Units for departments
* Group Policy enforcement
* Role-based access control

---

### Monitoring & Detection (Splunk)

* Centralized log collection from:

  * Windows Server
  * IT PCs
  * Marketing PCs
  * pfSense firewall
* SOC-lite visibility into:

  * Authentication events
  * Firewall blocks
  * Network activity
* Foundation for incident detection and response

---

### Remote Access (VPN)

* pfSense VPN for remote Marketing users
* Dedicated VPN subnet
* Firewall rules restricting VPN access to Marketing resources only
* AD-based authentication
* VPN connection logging for visibility and auditing

---

### DMZ (Public-Facing Services)

* Isolated VLAN for web/public servers
* Internet-facing access only to DMZ
* No direct access from DMZ to internal Server or LAN VLANs
* Controlled outbound access for updates and monitoring

---

## Security Principles Demonstrated

This project implements real-world enterprise and SMB security concepts:

* Network segmentation & VLAN isolation
* Least privilege access control
* Lateral movement prevention
* Centralized identity management
* Centralized logging and monitoring
* Defense-in-depth architecture
* Secure remote access via VPN
* Separation of internal and public-facing systems

---

## Project Structure

This documentation is organized into the following sections:

1. **Project Introduction & Architecture (This Section)**
2. Windows Server 2022 & Active Directory Deployment
3. pfSense Firewall & VLAN Network Architecture
4. Domain Join & Endpoint Integration
5. Firewall Rule Design & Security Policy Enforcement
6. Centralized Logging with Splunk
7. DMZ Design & Public-Facing Services
8. Backup Strategy & Disaster Recovery
9. VPN for Remote Users
10. Security Testing & Validation

---

## Why This Project Matters

This lab goes beyond basic home lab setups. It is designed to mirror how real SMBs:

* Secure internal networks
* Protect critical servers
* Support multiple departments
* Provide secure remote access
* Detect and respond to security events

This project demonstrates practical, hands-on experience in building and securing real infrastructure, not just theoretical knowledge.



# Section 2 — Infrastructure Installation & Network Configuration

## Windows Systems, Servers & pfSense Firewall Setup

## Overview

This section documents the deployment of the core infrastructure components for the Secure SMB Lab. This includes:

* Installation of pfSense firewall
* Installation of Windows Server and Windows client systems
* Installation of Ubuntu server (Splunk)
* Static IP configuration
* VLAN and interface assignment on pfSense
* Initial network connectivity and routing

At this stage, systems are **networked and segmented**, but not yet joined to Active Directory. Domain integration is covered in Section 3.

This mirrors real-world deployment where **network and security infrastructure is built first**, followed by identity and access integration.

---

## 1. pfSense Firewall Installation

### 1.1 pfSense Virtual Machine Deployment

A pfSense virtual machine was deployed to act as the central firewall and router for the environment.

Configuration:

* CPU: 2 vCPUs
* Memory: 2–4 GB RAM
* Disk: 20 GB
* Network Interfaces:

  * WAN (Internet-facing)
  * LAN (Internal trunk for VLANs)

pfSense is used as:

* Default gateway for all VLANs
* Inter-VLAN firewall enforcement
* NAT for Internet access
* VPN termination point
* Central network security control

---

### 1.2 pfSense Base Installation

Steps:

1. Booted VM from pfSense ISO
2. Completed pfSense installation
3. Assigned:

   * WAN interface to Internet-facing NIC
   * LAN interface to internal NIC
4. Set initial LAN IP for management access
5. Accessed pfSense Web UI for further configuration

---

## 2. VLAN & Interface Configuration on pfSense

To support network segmentation, multiple VLANs were created and assigned on the pfSense LAN interface.

### 2.1 VLAN Design

| VLAN Name      | Subnet          | Gateway IP   | Purpose               |
| -------------- | --------------- | ------------ | --------------------- |
| LAN (IT)       | 192.168.10.0/24 | 192.168.10.1 | IT workstations       |
| Server VLAN    | 192.168.20.0/24 | 192.168.20.1 | Servers               |
| Marketing VLAN | 192.168.30.0/24 | 192.168.30.1 | Marketing users       |
| DMZ VLAN       | 192.168.40.0/24 | 192.168.40.1 | Public-facing systems |
| VPN Subnet     | 192.168.50.0/24 | 192.168.50.1 | Remote users          |



<img width="1429" height="123" alt="aaaa1" src="https://github.com/user-attachments/assets/cecea28b-37ac-4952-941d-9087965c3b65" />
<img width="1231" height="136" alt="aaaa2" src="https://github.com/user-attachments/assets/0a9d67d6-5af9-4bee-a971-6781b62e9c95" />

---


### 2.2 VLAN Creation on pfSense

VLANs were created as subinterfaces on the LAN physical interface:

Steps:

1. Navigate to **Interfaces → Assignments → VLANs**
2. Create VLANs on LAN parent interface
3. Assign VLAN IDs for:

   * IT (LAN)
   * Server
   * Marketing
   * DMZ
4. Assign each VLAN to a pfSense interface
5. Enable each interface
6. Configure IPv4 Configuration Type as **Static IPv4**

---

### 2.3 Interface IP Address Assignment

Each VLAN interface was configured with a static gateway IP:

| Interface      | IP Address   |
| -------------- | ------------ |
| IT / LAN       | 192.168.10.1 |
| Server VLAN    | 192.168.20.1 |
| Marketing VLAN | 192.168.30.1 |
| DMZ VLAN       | 192.168.40.1 |
| VPN Interface  | 192.168.50.1 |

These IPs act as:

* Default gateway for hosts
* Inter-VLAN routing points
* Firewall enforcement boundaries

---

## 3. Windows Server Installation (Infrastructure Phase)

### 3.1 Windows Server VM Deployment

A Windows Server 2022 VM was deployed in the Server VLAN.

Configuration:

* CPU: 2 vCPUs
* Memory: 4–8 GB
* Disk: 60+ GB
* Network Adapter: Server VLAN (192.168.20.0/24)

---

### 3.2 Static IP Configuration (Pre-Domain)

The server was assigned a static IP before domain configuration:

* IP Address: 192.168.20.10
* Subnet Mask: 255.255.255.0
* Default Gateway: 192.168.20.1
* DNS Server: Temporarily set to itself (later finalized during AD setup)

This ensures reliable connectivity for infrastructure services.

---

## 4. Ubuntu Server Installation (Splunk Server)

An Ubuntu Server VM was deployed to host Splunk.

Configuration:

* VLAN: Server VLAN
* IP Address: 192.168.20.20
* Gateway: 192.168.20.1
* DNS: 192.168.20.10 (after AD setup)

At this stage, Ubuntu was configured with:

* Static IP
* Network connectivity to pfSense
* Access to Internet for package updates and Splunk installation
<img width="1507" height="827" alt="Screenshot 2026-01-24 154723" src="https://github.com/user-attachments/assets/02860a72-a074-4b1b-9d93-f6c4c4cd1cbd" />

---

## 5. Windows Client Installation (IT & Marketing)

### 5.1 IT Workstation

* VLAN: IT / LAN
* IP: 192.168.10.10
* Gateway: 192.168.10.1

Purpose:

* Administrative workstation
* Used for IT management and server access

---

### 5.2 Marketing Workstation

* VLAN: Marketing
* IP: 192.168.30.10
* Gateway: 192.168.30.1

Purpose:

* End-user simulation
* Restricted access to internal servers
* Used to test segmentation and security policies

---

## 6. Adding Systems to pfSense Interfaces

Each system VM was connected to the appropriate virtual network (VMnet / VLAN trunk) and mapped to the correct pfSense interface.

This ensured:

* Devices appeared on correct VLANs
* Traffic passed through pfSense for routing and firewall inspection
* Inter-VLAN routing occurred only through pfSense

Connectivity testing:

* Ping to pfSense gateway from each VLAN
* Ping between VLANs (initially allowed for validation)
* Internet access test from each VLAN

---

## 7. Why This Matters (Enterprise Context)

This phase reflects real-world infrastructure deployment:

* Network segmentation before identity integration
* Firewall placed as central routing and security device
* Servers isolated from user networks
* DMZ separated for public-facing services
* Dedicated subnet for future VPN users

This creates a secure foundation before joining systems to Active Directory and enforcing identity-based policies.


---

## ➡️ Section 3: Active Directory Domain Services (AD DS) Setup & Domain Integration

### 🎯 Objective
The goal of this section is to deploy Active Directory Domain Services (AD DS) on Windows Server 2022, create a centralized domain environment, and prepare the infrastructure for centralized authentication, authorization, and management of users and computers.

This allows the organization to:
- Centralize user authentication
- Enforce security policies using Group Policy Objects (GPOs)
- Manage users and computers from a single location
- Prepare for secure access to shared resources (SMB, file servers, etc.)

---

## 3.1 Domain Controller Setup (Windows Server 2022)

### Server Details
| Component        | Value               |
|------------------|---------------------|
| Server Role      | Domain Controller   |
| OS               | Windows Server 2022 |
| Server IP        | 192.168.20.10       |
| Network Segment  | Server LAN (192.168.20.0/24) |
| DNS Role         | Installed on DC     |

---

### Steps Performed

1. Installed **Active Directory Domain Services (AD DS)** role:
   - Server Manager → Add Roles and Features
   - Selected **Active Directory Domain Services**
   - Included required features
<img width="1010" height="830" alt="Screenshot 2026-01-20 195659" src="https://github.com/user-attachments/assets/d19c5dac-f892-449d-bb6f-fa6e2b3a44f1" />

2. Promoted server to a **Domain Controller**:
   - Selected **Add a new forest**
   - Created new domain:
     ```
     Domain Name: silas.local
     ```
   - Configured:
     - DNS Server
     - Global Catalog (GC)
     - Directory Services Restore Mode (DSRM) password
<img width="1033" height="864" alt="Screenshot 2026-01-20 195934" src="https://github.com/user-attachments/assets/6805514c-0e5a-4307-9b81-511f84e0591e" />

3. Server rebooted after promotion to complete domain controller setup

---

## 3.2 DNS Configuration

Active Directory relies heavily on DNS for authentication and service discovery.

### DNS Setup:
- DNS role installed automatically during DC promotion
- Domain controller acts as:
  - Primary DNS server for the environment
- All domain-joined machines will use:
  
<img width="816" height="693" alt="Screenshot 2026-01-20 205346" src="https://github.com/user-attachments/assets/22151a53-6735-4955-9cc4-3c3ffafbf2f4" />
<img width="759" height="532" alt="Screenshot 2026-01-20 205533" src="https://github.com/user-attachments/assets/473f3c41-87db-4f55-bd38-2e54b16fec15" />

Preferred DNS: 192.168.20.10




This ensures:
- Clients can locate the domain controller
- Kerberos authentication works properly
- Group Policy can be applied

---

## 3.3 Organizational Units (OU) Structure

To properly organize users and computers, Organizational Units (OUs) were created.

### OU Structure

```

silas.local
│
├── IT
│   ├── IT_Users
│
├── Marketing
│   ├── Marketing_Users
│
└── HR
└── Domain_Admins

```
<img width="818" height="589" alt="Screenshot 2026-01-20 211558" src="https://github.com/user-attachments/assets/a64962c8-7043-4139-a439-1a9559e96e2a" />


### Purpose of OUs:
- Logical separation of departments
- Easier application of Group Policies
- Delegation of administrative control
- Improved security management

---

## 3.4 User Account Creation

User accounts were created for different departments.

### Example Users

| Username        | Department  | OU              |
|----------------|-------------|-----------------|
| sly b      | IT          | IT      |
| it.user01      | IT          | IT       |
| mary jane     | Marketing   | Marketing|
| mkt user02     | Marketing   | Marketing|
| HR user02     | HR   | HR|

Each user account:
- Uses domain authentication
- Can be managed centrally
- Can be assigned Group Policies

---

## 3.5 Computer Account Preparation

At this stage, computers are **not yet joined to the domain** (this will be covered in Section 4).

However:
- OUs for computers were created in advance
- This allows proper placement when systems are joined later

---

## 3.6 Group Policy Planning (Pre-Configuration)

Group Policy Objects (GPOs) were planned to enforce:

- Restricted Control Panel access
- Security settings for standard users
- Password and account lockout policies
- Desktop and system restrictions

Actual GPO implementation will be applied after:
- Domain joining of workstations
- Department-based computer placement in OUs

(This will be covered in the next section.)

---

## 3.7 Security & Best Practices Applied

✔ Dedicated Server LAN for domain controller  
✔ Static IP assigned to DC  
✔ Centralized DNS for domain authentication  
✔ OU-based design for scalability  
✔ Department-based separation  
✔ Prepared for Group Policy enforcement  

---

## Result

At the end of this section:

✅ Active Directory Domain Services is fully deployed  
✅ Domain Controller is operational  
✅ DNS is correctly configured  
✅ OU structure is in place  
✅ User accounts are created  
✅ Environment is ready for domain joining and GPO enforcement  

## ➡️ Section 4: Domain Joining, Group Policy Configuration & Access Control

### 🎯 Objective
This section focuses on integrating workstations and servers into the Active Directory domain, applying Group Policy Objects (GPOs), and enforcing access control to simulate a real enterprise environment.

This enables:
- Centralized authentication
- Centralized policy enforcement
- Role-based access control
- Reduced administrative risk
- Standardized system configurations

---

## 4.1 Systems Joined to the Domain

The following systems were joined to the `silas.local` domain.

### Domain-Joined Systems

| System Name        | Role              | IP Address        | OU Placement              |
|--------------------|-------------------|-------------------|---------------------------|
| IT-PC           | IT Workstation    | 192.168.10.10     | IT_Computers              |
| Marketing-PC          | Marketing PC      | 192.168.30.10     | Marketing_Computers       |
| SRV-SPLUNK-01      | Splunk Server     | 192.168.20.20     | Server_Computers          |

---

## 4.2 DNS Configuration for Domain Join

To ensure successful domain joining, each system was configured with:

```

Preferred DNS Server: 192.168.20.10

```

This is critical because:
- AD authentication depends on DNS
- Domain controllers are located using DNS records
- Group Policy processing requires DNS resolution

Incorrect DNS settings will prevent domain join and authentication.

---

## 4.3 Domain Join Process

### Steps Performed on Each System

1. Open System Properties
2. Change computer name (if required)
3. Join domain:
<img width="927" height="537" alt="Screenshot 2026-01-20 212501" src="https://github.com/user-attachments/assets/c371c703-2ad4-43d8-a38d-95b0289c68c9" />
   
```

silas.local

```
4. Enter domain admin credentials
5. Reboot system
6. Verify domain login at Windows sign-in screen

---

## 4.4 Organizational Unit (OU) Assignment

After joining the domain, computer objects were moved to their correct OUs:

- IT workstations → `IT_Computers`
- Marketing workstations → `Marketing_Computers`
- Servers → `Server_Computers`

This ensures:
- Correct GPO targeting
- Department-based policy enforcement
- Logical asset organization

---

## 4.5 Group Policy Objects (GPOs) Implemented

### 4.5.1 Control Panel Restrictions (Standard Users)

To prevent unauthorized system changes, a GPO was applied to standard user OUs.

#### Policy Applied:
```

User Configuration
→ Administrative Templates
→ Control Panel
→ Prohibit access to Control Panel and PC Settings = Enabled

```

Effect:
- Standard users cannot access Control Panel
- Prevents system misconfiguration
- Reduces helpdesk incidents
- Enforces least privilege




---

### 4.5.2 Local Administrator Restrictions

Domain users were **not added** to local Administrators group.

Effect:
- Users cannot install unauthorized software
- Reduces malware risk
- Prevents privilege escalation

---

### 4.5.3 Password & Account Lockout Policies

Configured via Default Domain Policy:

- Minimum password length enforced
- Account lockout after multiple failed attempts
- Password complexity enabled

Effect:
- Reduces brute-force risk
- Improves domain security posture

---

## 4.6 Access Control Validation

### Tests Performed

| Test Case                          | Result |
|-----------------------------------|--------|
| Standard user opens Control Panel | ❌ Blocked |
| Standard user installs software  | ❌ Blocked |
| Admin user accesses Control Panel | ✅ Allowed |
| Domain login from IT PC           | ✅ Successful |
| Domain login from Marketing PC    | ✅ Successful |

---

## 4.7 Security & Enterprise Best Practices

✔ Least Privilege enforced  
✔ Department-based GPO scoping  
✔ Centralized authentication  
✔ DNS correctly integrated with AD  
✔ OU-driven policy targeting  
✔ Reduced attack surface on endpoints  

---

## Result

At the end of this section:

✅ All systems successfully joined to the domain  
✅ Department-based OUs populated  
✅ GPOs applied and enforced  
✅ User privileges restricted appropriately  
✅ Enterprise-style access control implemented  
<img width="1096" height="560" alt="Screenshot 2026-01-21 084403" src="https://github.com/user-attachments/assets/5a7ece36-e5e3-42e9-a6d7-968c0c9f87d6" />
<img width="794" height="570" alt="Screenshot 2026-01-21 083659" src="https://github.com/user-attachments/assets/fda3ae90-7ec6-4886-a2ec-606ac373bf4e" />
<img width="1088" height="531" alt="Screenshot 2026-01-21 083442" src="https://github.com/user-attachments/assets/ed86d83d-8ebf-49dd-9e56-b159ce307a70" />
<img width="754" height="529" alt="Screenshot 2026-01-21 083244" src="https://github.com/user-attachments/assets/ceaa2647-8e43-4243-b700-bb20399f5d23" />
<img width="516" height="405" alt="Screenshot 2026-01-21 083046" src="https://github.com/user-attachments/assets/3eeb7494-478b-4f6d-81fb-365cd0b8be04" />
<img width="927" height="537" alt="Screenshot 2026-01-20 212501" src="https://github.com/user-attachments/assets/fc6f92b4-6abf-4ac6-b504-656b6a0d505f" />
<img width="441" height="374" alt="Screenshot 2026-01-20 211940" src="https://github.com/user-attachments/assets/fc7c2232-4779-4df3-833b-babb185424e0" />
<img width="785" height="565" alt="Screenshot 2026-01-21 084727" src="https://github.com/user-attachments/assets/28f734f5-c7f1-4535-bd3f-ade332706e6e" />

 ---

## ➡️ Section 5: Firewall Rules, Network Segmentation & Zero Trust Enforcement

### 🎯 Objective
This section documents the firewall rule design and implementation on pfSense to enforce network segmentation, reduce lateral movement, and apply Zero Trust principles across all VLANs and subnets.

The goal is to ensure:
- Users only access what they need
- Servers are protected from workstations
- Compromised endpoints cannot move laterally
- High-value assets are isolated
- All access is explicitly defined and logged

---

## 5.1 Network Segmentation Overview

The environment is segmented into multiple security zones:

| Network        | Subnet            | Purpose                    |
|----------------|-------------------|----------------------------|
| IT LAN         | 192.168.10.0/24   | IT Administration          |
| Server LAN     | 192.168.20.0/24   | Domain & Infrastructure    |
| Marketing LAN  | 192.168.30.0/24   | User Workstations          |
| DMZ            | 192.168.40.0/24   | Public-Facing Services     |
| VPN Network    | 192.168.50.0/24   | Remote Users               |

Each zone has a different trust level and access policy.

---

## 5.2 Security Design Philosophy (Zero Trust)

Firewall policy follows these principles:

- **Default deny between segments**
- **Explicit allow rules only**
- **Least privilege access**
- **One-way trust where possible**
- **No workstation-to-server blanket access**
- **Logging enabled on security-relevant rules**

This prevents:
- Lateral movement
- Malware propagation
- Unauthorized access to servers
- Accidental exposure of sensitive systems

---

## 5.3 Interface-Based Rule Processing (pfSense Behavior)

pfSense evaluates rules:

- Top → Bottom
- First match wins
- Rules are applied on the **source interface**
- Traffic is filtered where it **enters** the firewall

This design means:
- Rules must be ordered carefully
- Broad rules go lower
- Specific rules go higher

---

## 5.4 IT LAN Firewall Rules (192.168.10.0/24)

### Purpose:
IT administrators require controlled access to infrastructure systems.

### Key Rules:
1. **Allow Marketing → AD-AUTH (DC)**
   - Destination: 192.168.20.10
   - Port: AD_SERVICES
   - Description: AD_AUTH
   - 
2. **Allow IT → Server LAN (Admin Access)**
   - Source: IT LAN net
   - Destination: Server LAN net
   - Description: IT admin access to servers

3. **Allow IT → Internet**
   - Source: IT LAN net
   - Destination: any
   - Description: Internet access
  
4. **Block IT → Other Internal VLANs**
   - Source: IT LAN net
   - Destination: RFC1918 networks
   - Description: Prevent unnecessary lateral access
     

<img width="1400" height="841" alt="Screenshot 2026-01-29 093503" src="https://github.com/user-attachments/assets/16028eb2-88df-4903-826e-245dc76373be" />

---

## 5.5 Marketing LAN Firewall Rules (192.168.30.0/24)

### Purpose:
User workstations should have minimal access.

### Key Rules:

1. **Allow Marketing → AD-AUTH (DC)**
   - Destination: 192.168.20.10
   - Port: AD_SERVICES
   - Description: AD_AUTH
   
<img width="820" height="769" alt="Screenshot 2026-01-26 134244" src="https://github.com/user-attachments/assets/47b02314-4060-447d-b066-d7786ec01592" />


1. **Allow Marketing → AD-AUTH (DC)**
   - Destination: 192.168.20.10
   - Port: AD_SERVICES
   - Description: AD_AUTH

4. **Allow Marketing → Internet**
   - Description: Web access

5. **Block Marketing → Server LAN (All Other)**
   - Destination: Server LAN net
   - Description: Prevent server probing

6. **Block Marketing → IT LAN**
   - Destination: IT LAN net
   - Description: Prevent peer access

---

## 5.6 Server LAN Firewall Rules (192.168.20.0/24)

### Purpose:
Servers should be protected and only initiate limited outbound connections.

### Key Rules:

1. **Allow Server → DNS**
   - Destination: Domain Controller
   - Port: 53

3. **Allow Server → Splunk (Logging)**
   - Destination: Splunk Server
   - Ports: Splunk Forwarder Ports

4. **Block Server → LAN & Marketing**
   - Destination: IT LAN & Marketing LAN
   - Description: Prevent reverse lateral movement

---

## 5.7 DMZ Firewall Rules (192.168.40.0/24)

### Purpose:
DMZ systems are treated as untrusted.

### Key Rules:

1. **Allow Internet → DMZ (Specific Services Only)**
   - Example: HTTPS to Web Server

2. **Allow DMZ → Server LAN (Logging Only)**
   - Destination: Splunk
   - Description: Centralized logging

3. **Block DMZ → Internal Networks**
   - Destination: RFC1918 networks
   - Description: Protect internal environment

---

## 5.8 VPN Firewall Rules (192.168.50.0/24)

### Purpose:
Remote users are treated like restricted internal users.

### Key Rules:

1. **Allow VPN → Marketing Resources**
   - Destination: Marketing File Shares

2. **Allow VPN → AD Authentication**
   - Destination: Domain Controller

3. **Block VPN → Server Admin Access**
   - Description: Prevent admin-level access

---

## 5.9 Logging & Monitoring Integration

Security-relevant rules were configured with logging enabled:

- Block rules
- DMZ rules
- Inter-VLAN access attempts

These logs are forwarded to Splunk for:

- Detection of lateral movement
- Policy violation alerts
- Suspicious access patterns
- Incident investigation

---

## Security Outcomes

✅ Lateral movement minimized  
✅ Servers isolated from users  
✅ Explicit access paths defined  
✅ DMZ treated as hostile zone  
✅ VPN users restricted  
✅ Zero Trust principles applied  
✅ Firewall rules aligned to business roles  

<img width="820" height="769" alt="Screenshot 2026-01-26 134244" src="https://github.com/user-attachments/assets/ef626ffc-e283-402d-b42a-c10b9def50d1" />
<img width="1660" height="868" alt="Screenshot 2026-01-26 134305" src="https://github.com/user-attachments/assets/0fb96fa2-6d8f-4cac-abff-d901265b3e68" />
<img width="1430" height="776" alt="Screenshot 2026-01-29 093541" src="https://github.com/user-attachments/assets/e1e90d83-5bcc-4885-9cf0-7fad0c4544e2" />
<img width="1432" height="790" alt="Screenshot 2026-01-29 093532" src="https://github.com/user-attachments/assets/3edf2fe9-d1c1-4000-9dc3-31891dd8394d" />
<img width="1425" height="775" alt="Screenshot 2026-01-29 093522" src="https://github.com/user-attachments/assets/866f6376-8cb5-4bb6-8104-ce3250143888" />
<img width="1400" height="841" alt="Screenshot 2026-01-29 093503" src="https://github.com/user-attachments/assets/1f07c6ff-8dab-484b-adde-1045361d388c" />
<img width="1429" height="785" alt="Screenshot 2026-01-29 093551" src="https://github.com/user-attachments/assets/8370a4b8-84e8-4e19-b346-db7ad3f31a1d" />

---

## Result

This firewall architecture enforces:

- Defense in depth
- Least privilege networking
- Enterprise-grade segmentation
- SOC-ready logging & detection
- Real-world firewall design methodology

---

➡️ **Next Section:**  
DMZ Architecture, Public Services & External Exposure Control


## ➡️ Section 6: Splunk Deployment, Universal Forwarders & Centralized Logging

### 🎯 Objective
This section documents the deployment of Splunk Enterprise and Universal Forwarders to provide centralized logging, security visibility, and detection capability across the entire SMB environment.

The goal is to:
- Centralize logs from servers, endpoints, and firewall
- Enable security monitoring and investigation
- Support incident response and threat detection
- Simulate real-world SOC and SIEM architecture

---

## 6.1 SIEM Architecture Overview

Splunk is deployed as a centralized Security Information and Event Management (SIEM) platform.

### Components:

| Component            | Role                         | IP Address        |
|----------------------|------------------------------|-------------------|
| Splunk Enterprise(ubuntu)    | SIEM / Log Indexer & Search  | 192.168.20.20    |
| pfSense Firewall     | Network Security Logs        | Firewall Syslog  |
| Windows Server 2022 | AD, DNS, Server Logs         | 192.168.20.10    |
| Marketing Workstation| User Endpoint Logs           | 192.168.30.10    |
| IT Workstation       | Admin Endpoint Logs          | 192.168.10.10    |

All systems forward logs to the centralized Splunk server.

---

## 6.2 Splunk Enterprise Installation (Ubuntu Server)

### Host:
- Ubuntu Server
- IP: 192.168.20.20
- Role: Central SIEM Server

### Installation Steps:

1. Download Splunk Enterprise for Linux
2. Install Splunk package
3. Start Splunk service:
```
   sudo /opt/splunk/bin/splunk start
````

4. Enable boot-start:

   ```bash
   sudo /opt/splunk/bin/splunk enable boot-start
   ```

5. Access Splunk Web:

   ```
   https://192.168.20.20:8000
   ```

6. Complete initial setup and create admin account
<img width="1436" height="947" alt="Screenshot 2026-01-28 154254" src="https://github.com/user-attachments/assets/0e3fdca9-1b09-4640-a421-c059d5ef76cb" />


---

## 6.3 Universal Forwarder Deployment (Windows Hosts)

Universal Forwarder installed on:

* Windows Server 2022 (Domain Controller)
* Marketing Workstation
* IT Workstation

### Purpose:

Forward Windows Event Logs and system data to Splunk.

### Installation Steps:

1. Download Splunk Universal Forwarder for Windows
2. Install with admin privileges
3. Configure deployment
4. installed sysmon(olafconfig)
5. configured the inputs.conf



### Enable Inputs:

* Windows Event Logs:

  * Security
  * System
  * Application

<img width="836" height="724" alt="Screenshot 2026-01-28 133412" src="https://github.com/user-attachments/assets/afd37abe-beba-406a-bef7-a9933faefd74" />
<img width="1426" height="643" alt="Screenshot 2026-01-28 134204" src="https://github.com/user-attachments/assets/590b5179-c795-4d44-a514-0a22e16f1b01" />
<img width="816" height="593" alt="Screenshot 2026-01-28 134428" src="https://github.com/user-attachments/assets/feb10839-e324-41fe-b219-9c13b8c6387c" />

<img width="1430" height="830" alt="Screenshot 2026-01-28 204506" src="https://github.com/user-attachments/assets/a0de09ef-c91c-480d-a034-f14efc6a7632" />
<img width="1919" height="867" alt="Screenshot 2026-01-28 203740" src="https://github.com/user-attachments/assets/c573810c-5e43-42eb-bb33-4e6bab42e466" />
<img width="1429" height="864" alt="Screenshot 2026-01-28 204709" src="https://github.com/user-attachments/assets/0e429bc8-9e75-420f-8e9d-9eea66b10f93" />


## 6.5 pfSense Firewall Log Forwarding (Syslog)

pfSense configured to forward logs via UDP Syslog.

### Configuration:

* Remote Syslog Server: `192.168.20.20`
* Port: `9969/UDP`
* Log Types:

  * Firewall
  * VPN
  * System
  * DHCP
  * DNS Resolver

### Purpose:

Capture:

* Blocked connections
* Inter-VLAN attempts
* VPN activity
* Suspicious traffic
<img width="1428" height="936" alt="Screenshot 2026-01-28 211041" src="https://github.com/user-attachments/assets/bc84ffec-d7af-4d87-8a95-f798b5719e5f" />

<img width="1433" height="866" alt="Screenshot 2026-01-29 085239" src="https://github.com/user-attachments/assets/3eab51e4-cd40-4711-9e36-562472741994" />
---





---

## 6.7 Data Sources & Visibility

### Log Types Collected:

| Source               | Log Types                     |
| -------------------- | ----------------------------- |
| Windows Server       | Auth, AD, DNS, System Events  |
| Windows Workstations | Logon, Security, Process Logs |
| Ubuntu Server        | Auth, Syslog, System Logs     |
| pfSense              | Firewall, VPN, Network Events |

---

## 6.8 Security Use Cases Enabled

This deployment supports:

* Brute force detection
* Lateral movement detection
* Firewall rule violations
* VPN login monitoring
* Unauthorized access attempts
* Malware activity indicators
* Authentication failures

---

## 6.9 Example Security Scenarios

### Scenario 1 — Failed Login Detection

Multiple failed logons from Marketing PC to DC trigger investigation.

### Scenario 2 — Lateral Movement Attempt

pfSense logs show blocked access from Marketing VLAN to Server VLAN.

### Scenario 3 — VPN Abuse

Remote VPN user attempts to access Server Admin ports.

---

## 6.10 SOC & Incident Response Readiness

Splunk enables:

* Centralized search
* Timeline reconstruction
* Event correlation
* Evidence collection
* Incident response workflow

---

## Security Outcomes

✅ Centralized visibility
✅ Endpoint + network telemetry
✅ SOC-style architecture
✅ Detection-ready environment
✅ Enterprise SIEM deployment
✅ Real-world monitoring capability

---

## Result

This Splunk deployment transforms the environment from basic IT into a **security-monitored enterprise network** with:

* Audit trails
* Threat detection
* Investigative capability
* Compliance-style logging

---


## 6.11 pfSense Firewall Syslog Integration with Splunk (UDP 9969)

### 🎯 Objective
To forward firewall, VPN, and system logs from pfSense to Splunk in real time for centralized security monitoring, detection, and investigation.

This allows Splunk to act as the single source of truth for network security events.

---

## Architecture Overview

pfSense acts as a primary network security control and generates critical security telemetry such as:

- Blocked connections
- Allowed connections
- Inter-VLAN traffic
- VPN activity
- DHCP and DNS activity
- System and service events

These logs are forwarded to Splunk using Syslog over UDP.

---

## Syslog Flow

```

pfSense Firewall
|
|  UDP Syslog (Port 9969)
v
Splunk Enterprise SIEM
(192.168.20.20)

```

---

## pfSense Configuration

### Location in pfSense:
Status → System Logs → Settings → Remote Logging

### Configuration Settings:

| Setting                | Value              |
|------------------------|--------------------|
| Remote Syslog Server   | 192.168.20.20      |
| Remote Syslog Port     | 9969               |
| Protocol               | UDP                |
| Log Format             | BSD (default)      |

### Log Types Enabled:

-Everything
<img width="1428" height="936" alt="Screenshot 2026-01-28 211041" src="https://github.com/user-attachments/assets/826063e9-195e-44a8-acd2-3fb785ba5481" />
<img width="1433" height="866" alt="Screenshot 2026-01-29 085239" src="https://github.com/user-attachments/assets/fa150896-a1be-42b4-aa65-9e8d8c380cfe" />

---

## Splunk Configuration (Receiving Syslog)

### UDP Data Input on Splunk

On the Splunk Enterprise server:

1. Navigate to:
   Settings → Data Inputs → UDP

2. Create New UDP Input:

| Setting        | Value          |
|----------------|----------------|
| Port           | 9969           |
| Source Type    | syslog / pfSense |
| Index          | pfsense_firewall |
| Host           | pfSense        |

---

## Firewall Rules to Allow Syslog Traffic

On pfSense Server LAN interface:

### Allow pfSense → Splunk Syslog

- Source: pfSense
- Destination: 192.168.20.20
- Protocol: UDP
- Port: 9969
- Description: Allow pfSense syslog to Splunk

This ensures firewall logs can reach the SIEM.

---

## Security Use Cases Enabled

This integration enables detection of:

- Blocked lateral movement attempts
- Scanning and probing activity
- Unauthorized access attempts
- VPN abuse
- Policy violations
- Suspicious outbound connections
- Internal threat behavior

---

## Operational & Security Benefits

✅ Real-time firewall visibility  
✅ Centralized network telemetry  
✅ Correlation with endpoint logs  
✅ SOC-style monitoring architecture  
✅ Improved incident response capability  

---

## Result

pfSense is fully integrated with Splunk, allowing:

- Network + endpoint correlation
- Enterprise-grade security monitoring
- Investigation of blocked and allowed traffic
- Audit trails for firewall activity
- Detection of suspicious internal and external behavior

This completes full-stack visibility across firewall, servers, and endpoints.




## ➡️ Section 7: Backups, Disaster Recovery & Business Continuity Strategy

### 🎯 Objective
This section documents the backup and disaster recovery (DR) strategy implemented to protect critical business systems, ensure data availability, and support business continuity in the event of:

- Hardware failure
- Ransomware attacks
- Accidental deletion
- System corruption
- Security incidents

The goal is to minimize downtime, prevent data loss, and ensure rapid recovery of business operations.

---

## 7.1 Business-Critical Systems Identified

The following systems were classified as critical to business operations:

| System              | Role                              | Priority |
|---------------------|-----------------------------------|----------|
| Windows Server 2022 | Domain Controller, DNS, File Server| HIGH     |
| Splunk Server       | SIEM & Security Monitoring        | HIGH     |
| File Shares         | Business Documents                | HIGH     |
| Active Directory    | Authentication & Identity         | CRITICAL |
| pfSense Config      | Network Security Configuration    | HIGH     |

---

## 7.2 Backup Architecture Overview

The environment follows a multi-layered backup strategy:

- Local server backups
- Offline backup copies
- Configuration exports
- Logical separation from production systems

This reduces the risk of single points of failure.

---

## 7.3 Windows Server Backup Configuration

### Tool:
- Windows Server Backup

### Backup Scope:

- System State (Active Directory, Registry, Boot files)
- File Shares
- Critical Volumes
- DNS Services

### Backup Type:
- Scheduled full backups
- Incremental backups between full backups

### Schedule Example:

| Backup Type | Frequency |
|-------------|-----------|
| Full Backup | Weekly    |
| Incremental| Daily     |

---

## 7.4 Active Directory Backup (System State)

System State backups protect:

- Active Directory Database (NTDS.dit)
- SYSVOL
- Group Policy Objects
- Registry
- Boot Configuration

This enables:

- Authoritative restore
- Non-authoritative restore
- Recovery from AD corruption
- Recovery from ransomware

---

## 7.5 File Server Backup Strategy

File shares are backed up to:

- Secondary backup storage
- Separate volume from production data

Features:

- Versioning
- Point-in-time restore
- Protection from accidental deletion
<img width="1428" height="896" alt="Screenshot 2026-01-29 092357" src="https://github.com/user-attachments/assets/00ad2214-c9d0-4d1f-85d7-947f0797a1e3" />
<img width="1431" height="891" alt="Screenshot 2026-01-29 092236" src="https://github.com/user-attachments/assets/b28a952a-adf2-41d4-a0f4-14e8e6c5b54f" />
<img width="1151" height="868" alt="Screenshot 2026-01-29 091030" src="https://github.com/user-attachments/assets/3731e1ca-9e1d-43f2-aa19-64aaaff27a17" />
<img width="1429" height="884" alt="Screenshot 2026-01-29 092532" src="https://github.com/user-attachments/assets/bf242df7-43de-4593-9900-5fa97d911461" />

---


