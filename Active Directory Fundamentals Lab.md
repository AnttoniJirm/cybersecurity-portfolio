# Active Directory Fundamentals Lab

**Platform:** Home Lab  Windows Server 2022  
**Date:** June 2026

---

## Objective
Build a functional Active Directory environment from scratch, simulating a real 
corporate Windows infrastructure with Domain Controller, users, groups, 
organizational units, and group policy.

---

## Environment
| Component | Details |
|-----------|---------|
| Domain Controller | Windows Server 2022 — DC01 |
| Domain | lab.local |
| SIEM | Wazuh 4.7.5 (Ubuntu 22.04 — 192.168.20.10) |
| Attacker | Kali Linux 2026.1 (192.168.20.11) |
| Network | Isolated internal network — labnet |

---

## Phase 1 — Installing Windows Server 2022

Downloaded Windows Server 2022 ISO directly from Microsoft Evaluation Center 
(180-day free trial). Created VM in VirtualBox with 4GB RAM, 2 CPUs, 50GB disk.

After installation, renamed the server via SConfig:

```powershell
# Option 2 in SConfig — renamed from WIN-XXXXXXX to DC01
```

---

## Phase 2 — Installing Active Directory Domain Services

```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
```

**What this does:** Installs the AD Domain Services role on Windows Server. 
Without this, the server is just a regular Windows machine. With it, the server 
can be promoted to a Domain Controller.

**Result:**

Success : True
Exit Code : Success
Feature Result : Active Directory Domain Services, Group Policy Management 

---

## Phase 3 — Promoting the Server to Domain Controller

```powershell
Install-ADDSForest -DomainName "lab.local" -DomainNetbiosName "LAB" -InstallDns
```

**What this does:** Promotes the server to Domain Controller and creates the 
`lab.local` domain. `-InstallDns` installs the DNS service alongside AD — 
Active Directory requires DNS to function. The server rebooted automatically 
after this command.

**Verification:**
```powershell
Get-ADDomain
```

**Result:** 

DNSRoot     : lab.local
NetBIOSName : LAB
PDCEmulator : DC01.lab.local 

---

## Phase 4 — Creating Users

```powershell
New-ADUser -Name "John Smith" -GivenName "John" -Surname "Smith" `
  -SamAccountName "jsmith" -UserPrincipalName "jsmith@lab.local" `
  -AccountPassword (ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force)
Enable-ADAccount -Identity "jsmith"

New-ADUser -Name "Jane Doe" -GivenName "Jane" -Surname "Doe" `
  -SamAccountName "jdoe" -UserPrincipalName "jdoe@lab.local" `
  -AccountPassword (ConvertTo-SecureString "P@ssw0rd123!" -AsPlainText -Force)
Enable-ADAccount -Identity "jdoe"
```

**What this does:** `New-ADUser` creates the user with name, username and domain 
email. `-AccountPassword` sets the password securely by converting plain text to 
a SecureString. `Enable-ADAccount` enables the account — by default users are 
created disabled.

**Verification:**
```powershell
Get-ADUser -Identity "jsmith"
```

**Result:** 

DistinguishedName : CN=John Smith,CN=Users,DC=lab,DC=local
Enabled           : True
SamAccountName    : jsmith
UserPrincipalName : jsmith@lab.local
SID               : S-1-5-21-2075389030-3758795886-1683061984-1103 

<img width="1026" height="770" alt="IT Team members" src="https://github.com/user-attachments/assets/6e297acb-8a29-4ef1-b886-591926d9fe09" />

---

## Phase 5 — Creating Security Group

```powershell
New-ADGroup -Name "IT-Team" -GroupScope Global -GroupCategory Security
Add-ADGroupMember -Identity "IT-Team" -Members "jsmith","jdoe"
```

**What this does:** Creates a Global Security group called IT-Team. 
`-GroupScope Global` means the group can be used across the entire forest. 
`-GroupCategory Security` means it controls access permissions, not just 
email distribution.

**Verification:**
```powershell
Get-ADGroupMember -Identity "IT-Team"
```

**Result:** 

name          : John Smith | SamAccountName : jsmith
name          : Jane Doe   | SamAccountName : jdoe 

---

## Phase 6 — Creating Organizational Unit and Moving Users

```powershell
New-ADOrganizationalUnit -Name "IT" -Path "DC=lab,DC=local"

Move-ADObject -Identity "CN=John Smith,CN=Users,DC=lab,DC=local" `
  -TargetPath "OU=IT,DC=lab,DC=local"
Move-ADObject -Identity "CN=Jane Doe,CN=Users,DC=lab,DC=local" `
  -TargetPath "OU=IT,DC=lab,DC=local"
```

**What this does:** Creates an OU called IT inside the domain — like a folder 
to organize users by department and apply specific policies. `Move-ADObject` 
moves users from the default `CN=Users` container into `OU=IT`. The `CN=` prefix 
means Common Name, `DC=` means Domain Component — this is how AD identifies 
objects by their location in the directory tree.

**Verification:**
```powershell
Get-ADUser -Filter * -SearchBase "OU=IT,DC=lab,DC=local"
```

**Result:** 

DistinguishedName : CN=John Smith,OU=IT,DC=lab,DC=local — Enabled: True
DistinguishedName : CN=Jane Doe,OU=IT,DC=lab,DC=local  — Enabled: True 

<img width="1033" height="781" alt="OU IT  O AD 003" src="https://github.com/user-attachments/assets/23a36a54-d7a0-404d-915c-d65028fdc22b" />

---

## Phase 7 — Creating and Applying Group Policy

```powershell
New-GPO -Name "Password-Policy" | New-GPLink -Target "OU=IT,DC=lab,DC=local"

Set-GPRegistryValue -Name "Password-Policy" `
  -Key "HKLM\System\CurrentControlSet\Services\Netlogon\Parameters" `
  -ValueName "MaximumPasswordAge" -Type DWord -Value 30
```

**What this does:** `New-GPO` creates a Group Policy Object called Password-Policy. 
The pipe `|` passes it directly to `New-GPLink` which links it to `OU=IT` — 
meaning the policy applies to all users and computers in that OU. 
`Set-GPRegistryValue` configures the policy to enforce password expiration every 
30 days, which is standard in corporate environments.

**Result:** 

DisplayName      : Password-Policy
DomainName       : lab.local
Owner            : LAB\Domain Admins
GpoStatus        : AllSettingsEnabled
ComputerVersion  : AD Version: 1, SysVol Version: 1
Target           : OU=IT,DC=lab,DC=local 

<img width="1007" height="783" alt="GPO criada e aplicada na OU=IT 004" src="https://github.com/user-attachments/assets/5892d306-a688-4393-9aef-1828c525e966" />


---

## AD Structure Summary 

lab.local
└── DC01.lab.local (Domain Controller)
└── OU=IT
├── jsmith (John Smith) — IT-Team member
├── jdoe (Jane Doe)     — IT-Team member
└── GPO: Password-Policy (MaximumPasswordAge: 30 days)

---

## Why This Matters for SOC

Active Directory is the identity backbone of most corporate Windows environments. 
Understanding AD structure is essential for SOC analysts because:

- Most attacks target AD — credential theft, privilege escalation, lateral movement
- User and group management is where access control begins
- Group Policy controls security configurations across the entire domain
- Unauthorized changes to AD objects (users, groups, GPOs) are high-priority alerts

---

## Lessons Learned

1. **AD requires DNS.** The `-InstallDns` flag is mandatory — without DNS, 
   domain authentication fails entirely.
2. **Users are disabled by default.** `New-ADUser` creates accounts in a disabled 
   state — `Enable-ADAccount` must be called separately.
3. **OU must exist before moving objects.** Attempting `Move-ADObject` before 
   creating the OU throws an error — order of operations matters.
4. **GPOs are empty by default.** Creating a GPO and linking it does nothing 
   until policies are explicitly configured inside it.

<img width="958" height="762" alt="final conf GPO" src="https://github.com/user-attachments/assets/b21f62e2-59f2-4d70-b13f-4802cb9aad72" />


---

## Tools Used
- Windows Server 2022
- PowerShell (AD DS module)
- VirtualBox 7.2
- Active Directory Domain Services
- Group Policy Management
