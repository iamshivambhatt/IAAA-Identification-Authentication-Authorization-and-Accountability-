# 01 — Identification

## Overview

Identification is the first pillar of IAAA: every user and device on the network must be assigned a **unique, distinct identity** before any authentication, authorization, or accounting can take place. In this lab, identification is implemented through Active Directory user objects, computer objects, and the Organizational Unit (OU) hierarchy that contains them.

## Domain Controller Details

| Property | Value |
|---|---|
| Host Name | SHIVAM01 |
| OS | Windows Server 2022 Datacenter |
| Domain | Shivam.local |
| Role | Primary Domain Controller (AD DS) |
| System Type | VMware20,1 (x64), 2 vCPU |

## OU Hierarchy

Queried with `dsquery ou -name *`:

```
"OU=Domain Controllers,DC=Shivam,DC=local"
"OU=Bhatt,DC=Shivam,DC=local"
"OU=Finance,OU=Bhatt,DC=Shivam,DC=local"
"OU=Information Technology,OU=Bhatt,DC=Shivam,DC=local"
"OU=Account Payable,OU=Finance,OU=Bhatt,DC=Shivam,DC=local"
"OU=Account Receivable,OU=Finance,OU=Bhatt,DC=Shivam,DC=local"
```

📷 *Evidence: `dsquery ou -name *` output — see `/screenshots/02.jpg`*

The Finance department is split into Account Payable and Account Receivable sub-OUs to allow granular GPO scoping and delegated administration later in the Authorization phase.

## User Identities

Queried with `dsquery user -name *`:

```
"CN=Administrator,CN=Users,DC=Shivam,DC=local"
"CN=Guest,CN=Users,DC=Shivam,DC=local"
"CN=krbtgt,CN=Users,DC=Shivam,DC=local"
"CN=Henry Ford,OU=Account Payable,OU=Finance,OU=Bhatt,DC=Shivam,DC=local"
"CN=Enzo Ferrari,OU=Account Payable,OU=Finance,OU=Bhatt,DC=Shivam,DC=local"
"CN=Ferdinand Porsche,OU=Account Receivable,OU=Finance,OU=Bhatt,DC=Shivam,DC=local"
"CN=Louis Chevrolet,OU=Account Receivable,OU=Finance,OU=Bhatt,DC=Shivam,DC=local"
"CN=Karl Probst,OU=Account Receivable,OU=Finance,OU=Bhatt,DC=Shivam,DC=local"
"CN=Shivam Bhatt,OU=Information Technology,OU=Bhatt,DC=Shivam,DC=local"
```

📷 *Evidence: `dsquery user -name *` output — see `/screenshots/01.jpg`*

Each user account is uniquely identified by its Distinguished Name (DN), guaranteeing no two identities can collide within the directory, even across different OUs.

## Computer Identities

Queried with `dsquery computer -name *`:

```
"CN=SHIVAM01,OU=Domain Controllers,DC=Shivam,DC=local"
"CN=CLIENT01,CN=Computers,DC=Shivam,DC=local"
"CN=CLIENT02,CN=Computers,DC=Shivam,DC=local"
```

📷 *Evidence: `dsquery computer -name *` output — see `/screenshots/03.jpg`*

Machine identities are provisioned the same way as user identities — every domain-joined device receives its own AD object and DN, allowing it to be authenticated and governed independently of the user logged into it.

## Key Takeaway

Identification in this lab is enforced structurally: the OU design groups identities logically by department/function (Finance vs. IT), while `dsquery` confirms every user and computer object carries a globally unique DN within `Shivam.local`. This foundation is what makes the Authentication and Authorization pillars (covered next) possible.

---
⬅️ [Back to project overview](../README.md) | ➡️ [Next: Authentication](../02-Authentication)
