# Active-Directory-Identity-Governance-Hardened-Security-Visibility-Lab


## 📌 Project Architecture Overview
This production-grade Identity & Access Management (IAM) and Identity Governance (IGA) laboratory simulates an enterprise workforce environment. Built using **Windows Server 2022** and **Windows 10 Pro** hypervised within a localized virtualization stack, this lab acts as a proving ground for centralizing directory services, hardening access control, and automating identity lifecycles.

Rather than relying on default configurations, the depth of this implementation focuses on building a secure **Windows Event Forwarding (WEF)** log transport pipeline over encrypted channels alongside programmatic **Joiner, Mover, and Leaver (JML)** automation.

---

## 🎯 Core Engineering Objectives
* **Directory Services Deployment:** Provision and harden Active Directory Domain Services (AD DS) and integrated DNS infrastructure.
* **Cryptographic Log Pipeline Architecture:** Engineer a centralized Windows Event Collector (WEC) leveraging a Source-Initiated Push model over mutually authenticated WinRM HTTPS (Port 5986).
* **Identity Lifecycle Governance Automation:** Develop error-resistant PowerShell Core scripts to execute programmatic JML workflows, minimizing manual administration overhead.
* **Access Control Layer Enforcement:** Design and map structural Role-Based Access Control (RBAC) schemas adhering strictly to the Principle of Least Privilege (PoLP).
* **Enterprise Telemetry Auditing:** Enforce Advanced Audit Group Policy Objects (GPOs) to extract SIEM-ready Indicators of Compromise (IoCs).

---

## 💻 Infrastructure & System Specifications

```text
                     [ iamlab.local Domain ]
                     
+-----------------------+           +-----------------------+
|    WIN-A19BKD6L34B    |           |     WIN10-CLIENT      |
|  Windows Server 2022  | <=======  |    Windows 10 Pro     |
|   Domain Controller   |  WinRM    |  Workstation Endpoint |
|  (WEC / AD DS / DNS)  |  HTTPS    |  (Telemetry Source)   |
+-----------------------+  (5986)   +-----------------------+
```

### 🎛️ Domain Controller (`WIN-A19BKD6L34B`)
* **Operating System:** Windows Server 2022 Standard
* **Roles:** Active Directory Domain Services (AD DS), Domain Name System (DNS), Windows Event Collector (WEC)
* **Resource Profile:** 4 GB RAM | 17 GB Provisioned Storage

### 🖥️ Client Workstation (`WIN10-CLIENT`)
* **Operating System:** Windows 10 Pro (Domain-Joined)
* **Identity Target:** Telemetry Endpoint & Workforce Simulation Host
* **Resource Profile:** 4 GB RAM | 12 GB Provisioned Storage

### 🔌 Virtualization & Core Drivers
* **Platform:** UTM Hypervisor (SPICE Guest Tools)
* **Network & Storage Adapters:** VirtIO Paravirtualized Drivers
* **Topology:** Isolated NAT Networking

---

## 🛠️ Advanced Engineering Troubleshooting & Diagnostics Log
The core technical validity of this project is demonstrated by the systematic remediation of multi-layered enterprise protocol blockers encountered during deployment:

### 1. WinRM HTTPS Listener Pipeline & Cryptographic Binding

* **The Problem:** Direct network drops (`TcpTestSucceeded : False`) and unencrypted transport flags on Port 5985 exposed log telemetry to potential credential sniffing.
* **The Triage:** Taped and removed default HTTP listeners. Generated dedicated cryptographic self-signed certificates, captured the explicit thumbprint, and bound it directly to WinRM HTTPS Port 5986. Opened target network profiles using localized firewall constraints:

   ```cmd
  netsh advfirewall firewall add rule name="WinRM HTTPS Port 5986" dir=in action=allow protocol=TCP localport=5986

### 2. Kerberos Clock-Skew Policy Failures (W32Time Engine)

* **The Problem: Hypervisor state suspension caused severe time drift on the client workstation. This broke Kerberos authentication tokens, resulting in total Group Policy blockages (gpupdate /force errors) and pipeline drops.
* **The Triage: Performed a hard reset of the localized time subsystem tracking database. Flushed and unregistered corrupted configuration keys before forcing cross-domain sync with the root Domain Controller:


   ```cmd
   DOS
   net stop w32time
   w32tm /unregister
   w32tm /register
   net start w32time
   w32tm /resync /rediscover


###3. WEC Access Control Layer Exceptions (Client-Side Event 102)

*The Problem: The client engine successfully processed the subscription handshake (Event ID 100) but was blocked from reading or forwarding low-level security log channels, generating permission faults.

*The Triage: Identified token evaluation limitations on the local machine transfer agent (NT AUTHORITY\NETWORK SERVICE). Remedied this access gap by injecting the system account directly into the local built-in Event Log Readers container, allowing the pipeline to stream events.


⚡ Automated Identity Governance Engines (PowerShell Core)
#To demonstrate production scalability, the manual management of objects within Active Directory Users and Computers (ADUC) was replaced with single-line parameter pipeline scripts.

🚀 1. The Programmatic Joiner Protocol (User Provisioning)Handles employee onboarding, forcing plain-text strings into secure tokens, applying standard organizational unit indexing, and mapping group-based authorization scopes:


PowerShell
   ```cmd
    #---CONFIGURATION VARIABLES---
    $NewUser   = "JohnDoe"
    $UserGroup = "Event Log Readers"
    $Domain    = "iamlab.local"
    $Password  = ConvertTo-SecureString "CyberLab2026!" -AsPlainText -Force

    #Provision Active Directory Security Principal Object
    New-ADUser -Name $NewUser -SamAccountName $NewUser -UserPrincipalName "$NewUser@$Domain" -AccountPassword $Password -      ChangePasswordAtLogon $false -Enabled $true -Path "CN=Users,DC=iamlab,DC=local"

    #Align Access Token to Targeted Security Boundary Group
    Add-ADGroupMember -Identity $UserGroup -Members $NewUser
   ```

🚨 2. The Programmatic Leaver Protocol (Offboarding & Isolation)Instantly revokes account authentication and token-signing capabilities, changing the active status flags and isolating the identity object to a dedicated containment Organizational Unit:


PowerShell
   ```cmd
    #--- CONFIGURATION VARIABLES ---
    $TargetUser = "JohnDoe"
    $TargetOU   = "OU=Disabled_Users,DC=iamlab,DC=local"

    #1. Immediate Account Status Token Revocation
    Disable-ADAccount -Identity $TargetUser

    #2. Structural Object Relocation to Containment Isolation OU
    Get-ADUser -Identity $TargetUser | Move-ADObject -TargetPath $TargetOU
   ```

---

📊 Directory Architecture & Access Governance Schema
*🏢 Organizational Unit (OU) Tree

```text
iamlab.local (Root Domain)
├── 📂 Admins          (Privileged Engineering Identities)
├── 📂 Users           (Standard Workforce Personas)
├── 📂 Disabled_Users  (Containment & Offboarding Isolation Zone)
└── 📂 Workstations    (Machine Objects)
```

---

👥 Role-Based Access Control (RBAC) Matrix
Permissions are decoupled from individual users and mapped to security groups to ensure deterministic access management.

| Functional Persona | Security Group | Default Access Mapping | Account Examples |
| :--- | :--- | :--- | :--- |
| **IT Administration** | `IT_Group` | Domain Engine Alterations / Full Escalation | `AdminUser`, `IT_User` |
| **Human Resources** | `HR_Group` | Read/Write Access to Personnel Object Contexts | `HR_User` |
| **Corporate Finance** | `Finance_Group` | Isolated Accounting Network Resource Shares | `Finance_User` |


---

🔍 Security Visibility & Monitored TelemetryWith advanced audit policies enforced via domain GPOs, the encrypted log pipeline captures SIEM-ready security audit data:


🔬 Verified Indicators of Compromise (IoCs) Tracked
I. Brute-Force / Unauthorized Access Auditing (Event ID 4625)
Captures failed authentication metadata, providing defense analysts with explicit target attribution (MaliciousActor) and source origin tracking (WIN10-CLIENT):

```text
Plaintext

Log Name:      Security
Event ID:      4625
Keywords:      Audit Failure
Computer:      WIN-A19BKD6L34B.iamlab.local (Forwarded from Win10-Client)
Description:   An account failed to log on.
Account Name:  MaliciousActor
Logon Type:    2 (Interactive)
Status:        0xC000006A (Incorrect Password Entered)
```

II. Automated Security Policy Account Lockouts (Event ID 4740)
Fires automatically when password failure thresholds are breached, locking out the user and recording the exact workstation that generated the lockout sequence:

```text
Plaintext

Log Name:      Security
Event ID:      4740
Keywords:      Audit Success
Description:   A user account was locked out.
Target Account: IAMLAB\JohnDoe
Caller Computer Name: WIN10-CLIENT
```

III. Privileged Group Membership Mutations (Event ID 4728 / 4729)
Monitors privilege escalation vectors by logging whenever an account is added to or removed from highly sensitive infrastructure groups like Domain Admins.

---
🧰 Technical Competencies Demonstrated
* Directory Infrastructure Engine Architecture: Active Directory Domain Services (AD DS) & Integrated DNS Management.
* Identity Governance & Administration (IGA): Programmatic JML Lifecycle Engineering & Provisioning.
* Defensive Engineering Pipelines: Windows Event Forwarding (WEF) & Centralized Windows Event Collection (WEC).
* Secure Remote Transport Engineering: WinRM Architecture Hardening, Port 5986 Configuration, & TLS/SSL Certificate Management.
* Scripted System Automation: Advanced PowerShell Scripting & Core Parameter Optimization.
* Access Control Framework Design: Role-Based Access Control (RBAC) Schema & Principle of Least Privilege (PoLP) Enforcement.
