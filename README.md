# IAAA Lab — Identity, Authentication, Authorization & Accounting on Windows Server AD

A self-built enterprise-style Active Directory environment demonstrating the four pillars of identity and access security — **Identification, Authentication, Authorization, and Accounting (IAAA)** — end to end, with live evidence for every stage.

The lab simulates a small organization ("Bhatt Corp") with a Finance department split into **Accounts Payable** and **Accounts Receivable** teams, and proves — with Event Viewer logs, not just configuration screenshots — that each team can only access its own data.

---

## 📐 Architecture

```
Shivam.local (Domain)
│
├── SHIVAM01 — Windows Server 2022 Datacenter (Primary Domain Controller)
├── CLIENT01, CLIENT02 — Domain-joined Windows clients
└── bhatt01 — Ubuntu 24.04.3 LTS client (cross-platform access)

OU Structure:
Shivam.local
└── Bhatt
    ├── Information Technology        (Shivam Bhatt)
    └── Finance
        ├── Account Payable           (Henry Ford, Enzo Ferrari)
        └── Account Receivable        (Ferdinand Porsche, Louis Chevrolet, Karl Probst)

Security Groups (RBAC):
├── AP Group  → Full Control on E:\Finance\Accounts Payable
└── AR Group  → Read-only on   E:\Finance\Accounts Receivable
```

| Component | Details |
|---|---|
| Domain Controller | `SHIVAM01` — Windows Server 2022 Datacenter |
| Domain | `Shivam.local` |
| Platform | VMware (VMware20,1), 2 vCPU |
| Windows Clients | `CLIENT01`, `CLIENT02` |
| Linux Client | `bhatt01` — Ubuntu 24.04.3 LTS |

---

## 📋 Table of Contents
1. [Identification](#1-identification)
2. [Authentication](#2-authentication)
3. [Authorization](#3-authorization)
4. [Accounting / Auditing](#4-accounting--auditing)
5. [Key Result: Proving RBAC Actually Works](#-key-result-proving-rbac-actually-works)
6. [Skills Demonstrated](#️-skills-demonstrated)

---

## 1. Identification

Every user and device must first be assigned a **unique, distinct identity** before anything else in IAAA can happen. This lab enforces identification through the AD object hierarchy.

**Organizational Units** (`dsquery ou -name *`):
```
OU=Domain Controllers,DC=Shivam,DC=local
OU=Bhatt,DC=Shivam,DC=local
OU=Finance,OU=Bhatt,DC=Shivam,DC=local
OU=Information Technology,OU=Bhatt,DC=Shivam,DC=local
OU=Account Payable,OU=Finance,OU=Bhatt,DC=Shivam,DC=local
OU=Account Receivable,OU=Finance,OU=Bhatt,DC=Shivam,DC=local
```
📷 `screenshots/01-dsquery-ous.jpg`

**User objects** (`dsquery user -name *`) — 6 named identities plus built-in accounts, each carrying a unique Distinguished Name:
```
CN=Henry Ford,OU=Account Payable,OU=Finance,OU=Bhatt,DC=Shivam,DC=local
CN=Enzo Ferrari,OU=Account Payable,OU=Finance,OU=Bhatt,DC=Shivam,DC=local
CN=Ferdinand Porsche,OU=Account Receivable,OU=Finance,OU=Bhatt,DC=Shivam,DC=local
CN=Louis Chevrolet,OU=Account Receivable,OU=Finance,OU=Bhatt,DC=Shivam,DC=local
CN=Karl Probst,OU=Account Receivable,OU=Finance,OU=Bhatt,DC=Shivam,DC=local
CN=Shivam Bhatt,OU=Information Technology,OU=Bhatt,DC=Shivam,DC=local
```
📷 `screenshots/02-dsquery-users.jpg`

**Computer objects** (`dsquery computer -name *`):
```
CN=SHIVAM01,OU=Domain Controllers,DC=Shivam,DC=local
CN=CLIENT01,CN=Computers,DC=Shivam,DC=local
CN=CLIENT02,CN=Computers,DC=Shivam,DC=local
```
📷 `screenshots/03-dsquery-computers.jpg`

---

## 2. Authentication

Authentication verifies that an identity is who it claims to be, before granting access to a session.

**Cross-platform login** — `bhatt01`, an Ubuntu 24.04.3 LTS machine, demonstrates authentication outside the pure Windows stack, with the `shivamb` account logging in and passing credential verification before reaching a shell:
```
bhatt01 login: shivamb
Password:
Welcome to Ubuntu 24.04.3 LTS (GNU/Linux 6.8.0-84-generic x86_64)
```
📷 `screenshots/08-linux-authentication.jpg`

**Group Policy processing on logon** — the RSOP (Resultant Set of Policy) report captured for `CLIENT01\Kaka` confirms that domain authentication triggers policy evaluation at logon, one of the checks that ties a successful authentication event to enforced domain configuration.
📄 `Report.html` (full RSOP report)

---

## 3. Authorization

Authorization determines *what* an authenticated identity is allowed to do. This lab implements it with **NTFS permissions scoped to security groups** — textbook RBAC — rather than assigning permissions to individual users.

**NTFS folder permissions:**

| Folder | Group | Permissions |
|---|---|---|
| `E:\Finance\Accounts Payable` | AP Group | Full Control, Modify, Read & Execute, Write |
| `E:\Finance\Accounts Receivable` | AR Group | Read only |

📷 `screenshots/04-ntfs-permissions-accounts-payable.jpg`
📷 `screenshots/05-ntfs-permissions-accounts-receivable.jpg`

**Security group membership (RBAC):**

| Group | Members |
|---|---|
| AP Group | Enzo Ferrari, Henry Ford |
| AR Group | Ferdinand Porsche, Karl Probst, Louis Chevrolet |

📷 `screenshots/06-ap-group-members.jpg`
📷 `screenshots/07-ar-group-members.jpg`

Access is granted purely by **group membership**, not by individually assigning rights per user — meaning onboarding a new Finance hire is as simple as adding them to the right group, with no folder-level reconfiguration.

---

## 4. Accounting / Auditing

Accounting is the pillar most projects skip — this lab doesn't. Every access attempt to the Finance folders is logged via a Group Policy **Audit Object Access** policy, capturing both **Success** and **Failure** events.

**Audit policy configuration** — Default Domain Policy, Local Policies → Audit Policy → Audit object access, with both Success and Failure auditing enabled:
📷 `screenshots/09-audit-policy-config.jpg`

**Event Viewer evidence (Security log):**

| Event ID | User | Action | Result |
|---|---|---|---|
| 4663 | Henry Ford (AP Group) | Accessed `E:\Finance\Accounts Payable` | ✅ Audit Success |
| 4656 | Henry Ford (AP Group) | Attempted `E:\Finance\Accounts Receivable` | ❌ Audit Failure |
| 4663 | Louis Chevrolet (AR Group) | Accessed `E:\Finance\Accounts Receivable` | ✅ Audit Success |
| 4656 | Louis Chevrolet (AR Group) | Attempted `E:\Finance\Accounts Payable` | ❌ Audit Failure |

📷 `screenshots/10-audit-success-henryford-ap.jpg`
📷 `screenshots/11-audit-failure-henryford-ar.jpg`
📷 `screenshots/12-audit-success-louischevrolet-ar.jpg`
📷 `screenshots/13-audit-failure-louischevrolet-ap.jpg`

---

## 🏆 Key Result: Proving RBAC Actually Works

Most AD labs stop at showing permission configuration. This one goes a step further: the Event Viewer logs above **prove enforcement**, not just intent.

- **Henry Ford** (AP Group) can read/write his own department's folder (Event 4663 – Success) but is **denied and logged** the moment he tries to touch Accounts Receivable (Event 4656 – Failure).
- **Louis Chevrolet** (AR Group) shows the exact mirror image — access granted on his own folder, denied and logged on the other team's folder.

This closes the loop across all four IAAA pillars: identities were provisioned (Identification), verified at logon (Authentication), restricted by group-scoped NTFS permissions (Authorization), and every access attempt — allowed or denied — was captured in the Security event log (Accounting).

---

## 🛠️ Skills Demonstrated

- Active Directory Domain Services (AD DS) deployment and OU design
- Role-Based Access Control (RBAC) via security groups
- NTFS permission scoping and delegation
- Group Policy Object (GPO) creation and Audit Policy configuration
- Windows Security Event Log analysis (Event IDs 4656, 4663, 5145)
- Group Policy Result (RSOP) reporting
- Cross-platform (Windows + Linux) domain environment administration
- Command-line AD querying (`dsquery`)

---

## 📁 Suggested Repo Structure

```
IAAA-Lab/
├── README.md                 ← this file
├── Report.html                ← full GPO RSOP report
└── screenshots/
    ├── 01-dsquery-ous.jpg
    ├── 02-dsquery-users.jpg
    ├── 03-dsquery-computers.jpg
    ├── 04-ntfs-permissions-accounts-payable.jpg
    ├── 05-ntfs-permissions-accounts-receivable.jpg
    ├── 06-ap-group-members.jpg
    ├── 07-ar-group-members.jpg
    ├── 08-linux-authentication.jpg
    ├── 09-audit-policy-config.jpg
    ├── 10-audit-success-henryford-ap.jpg
    ├── 11-audit-failure-henryford-ar.jpg
    ├── 12-audit-success-louischevrolet-ar.jpg
    └── 13-audit-failure-louischevrolet-ap.jpg
```

📌 **Status:** Complete — Identification, Authentication, Authorization, and Accounting all implemented and evidenced with live logs.
