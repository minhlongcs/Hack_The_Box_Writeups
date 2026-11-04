#sherlock #harden #active-directory #purple-knight #kerberos #windows-server
Difficulty: Easy
Categories: Harden
Description: You are a security engineer tasked with scanning and remediating key vulnerabilities in Forela's Active Directory domain using **Semperis Purple Knight**. Remediating the finding in each task makes the corresponding flag appear on the Desktop.

- Personal Summary:
    - New terms, things to learn: Semperis Purple Knight, Fine-Grained Password Policy (Password Settings Object), LDAP signing, MITRE ATT&CK / D3FEND mapping, `ms-DS-MachineAccountQuota`, Kerberoasting and SPNs, group Managed Service Accounts (gMSA), `msDS-SupportedEncryptionTypes`, AS-REP Roasting, unconstrained vs constrained delegation, DCSync rights
    - Goals: Run a full Purple Knight scan of `forela.local`, answer the questions from the report, and remediate the riskier findings to collect the flags.

> [!summary] TL;DR: findings and fixes
> | Task | Finding | Answer / Fix |
> |---|---|---|
> | 1 | Indicators evaluated | **116** |
> | 2 | Privileged Users with Weak Password Policy | **4** users |
> | 3 | Weak password length | Fine-Grained Password Policy, min length 15, applied to `Domain Users` |
> | 4 | LDAP signing not required | First ATT&CK tactic: **Privilege Escalation** |
> | 5 | LDAP signing not required | GPO: *LDAP server signing requirements* → **Require signing** |
> | 6 | Privileged accounts, password never expires | D3FEND ID: **D3-SPP** |
> | 7 | Password never expires | Untick it on `analyst`, `svc_checker`, `Administrator` |
> | 8 | Unprivileged users can add computers | Attribute: **ms-DS-MachineAccountQuota** |
> | 9 | Unprivileged users can add computers | Set `ms-DS-MachineAccountQuota` = **0** |
> | 10 | Privileged users with SPN | **BackupSvc/DC01.forela.local** |
> | 11 | gMSA not in use | **Windows Server 2016** |
> | 12 | SPNs supporting RC4 | **4** |
> | 13 | Kerberoasting | Remove SPNs, migrate `svc_iis` to a gMSA, AES-only for `svc_sql` |
> | 14 | Kerberos pre-auth disabled | **ASREP-Roasting** |
> | 15 | Kerberos pre-auth disabled | Re-enable pre-auth on `svc_backup` |
> | 16 | Unconstrained delegation | **CN=WEBSERVER01,CN=Computers,DC=forela,DC=local** |
> | 17 | Unconstrained delegation | Constrain `WEBSERVER01` to `MSSQLSvc/db.forela.local:1433` |
> | 18 | Non-default DCSync rights | **FORELA\Auditors** |
> | 19 | Non-default DCSync rights | Remove replication rights from `FORELA\Auditors` |

---

## Scenario

You are a security engineer tasked with scanning and remediating key vulnerabilities in Forela's Active Directory domain using Semperis Purple Knight.

To solve this Sherlock follow the below steps:

- From the Desktop, right-click and run Purple Knight as Administrator. When prompted enter the password `H4ck_The_Blue_!`
- Perform a full scan of the `forela.local` Active Directory Forest
- When you remediate the finding listed in a task, the corresponding flag will appear on the desktop. This can sometimes take up to a minute to show.
- When complete you can optionally rescan the environment to see the improvement in score. Remember to relaunch PurpleKnight as Administrator to ensure you have full access.

> [!note]
> While the objective of the Sherlock is to get as clean a Purple Knight report as possible, as objects in the Active Directory age, it is possible that new findings that are not listed in the tasks will appear in the report. You can also attempt to remediate these and rescan the environment should you wish.

## About

As part of the HTB partnership with Semperis this is an opportunity for players to assess Active Directory for vulnerabilities using the Purple Knight tool, and perform remediation actions against some of the riskier findings.

Credentials for Purple Knight:
![](../Pasted%20image%2020261103214749.png)
![](../Pasted%20image%2020261103224912.png)

---

## Setup

### Getting Purple Knight
- Download page: https://www.semperis.com/purple-knight/
- You have to fill out a form to get the download link. As a university student, you can use your university email for it.
- Remember to **run as Administrator** for maximum access.

### Running the scan
1. RDP into the given desktop with the given credentials (username: `FORELA\analyst`):
   ![](../Pasted%20image%2020261103224444.png)
2. Run PurpleKnight as Administrator on the RDP machine:
   ![](../Pasted%20image%2020261103225646.png)
3. Enter `forela.local` as the Active Directory forest to assess:
   ![](../Pasted%20image%2020261103231058.png)
4. Click **Run Tests**:
   ![](../Pasted%20image%2020261104001043.png)
5. When the scan finishes, view the results:
   ![](../Pasted%20image%2020261104024752.png)
6. Open the full PDF report, which is where most answers come from:
   ![](../Pasted%20image%2020261104030250.png)

---

## Doing the sherlock

### Task 1: How many Indicators were evaluated as part of the full scan?

![](../Pasted%20image%2020261104030429.png)

- Answer: 116

### Task 2: How many users were identified as being vulnerable to the "Privileged Users with Weak Password Policy" finding?

![](../Pasted%20image%2020261104030901.png)

- Answer: 4

### Task 3: Enforce a modern minimum password length of 15 characters for standard users by implementing a Fine-Grained Password Policy on the `Domain Users` group.

- Microsoft guide: https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/adac/fine-grained-password-policies?tabs=adac
  ![](../Pasted%20image%2020261104102203.png)
- I used **Active Directory Administrative Center**:
  ![](../Pasted%20image%2020261104103012.png)
1. Navigate to **forela (local)** → **System** → **Password Settings Container**.
2. In the **Tasks** pane, choose **New** → **Password Settings**.
3. Fill in the property page to create a new **Password Settings** object. **Name** and **Precedence** are required.
   ![](../Pasted%20image%2020261104103522.png)
4. Set **Minimum password length** to `15`, and under **Directly Applies To** add `Domain Users`.

- Flag:
  ![](../Pasted%20image%2020261104103841.png)

### Task 4: Review the finding "LDAP signing is not required on Domain Controllers" and the Security Frameworks subsection. What is the first MITRE ATT&CK Tactic listed?

![](../Pasted%20image%2020261104103949.png)

- Answer: Privilege Escalation

### Task 5: Configure the Domain Controller to enforce mandatory LDAP signing and reject unauthenticated or unsigned LDAP bind requests.

This protects the domain from NTLM Relay and Machine-In-The-Middle attacks.

- Microsoft guide: https://learn.microsoft.com/en-us/windows-server/identity/manage-ldap-signing-group-policy?tabs=windows-server-2025
1. Open the **Group Policy Management Console** (`gpmc.msc` from Run, or from the Start Menu).
2. Expand **Forest: forela.local** to find **Default Domain Controllers Policy** → right-click → **Edit**.
   ![](../Pasted%20image%2020261104105934.png)
3. Go to **Default Domain Controllers Policy** → **Computer Configuration** → **Policies** → **Windows Settings** → **Security Settings** → **Local Policies** → **Security Options**.
4. Right-click **Domain controller: LDAP server signing requirements enforcement** → **Properties** → set to **Require Signing**.
   ![](../Pasted%20image%2020261104110416.png)

- Flag (after changing the policy setting):
  ![](../Pasted%20image%2020261104110847.png)

### Task 6: Review the "Privileged accounts with a password that never expires" finding. According to PurpleKnight, what is the ID associated with the MITRE D3FEND Technique listed?

![](../Pasted%20image%2020261104110945.png)
![](../Pasted%20image%2020261104111037.png)

- Answer: D3-SPP

### Task 7: Ensure standard password rotation policies apply to all accounts by removing any explicit 'Password never expires' properties.

1. The Security Indicator lists two affected accounts: `analyst` and `svc_checker`.
2. Open **Active Directory Users and Computers** → **forela.local** → **Users**.
   ![](../Pasted%20image%2020261104113728.png)
3. For `analyst` and `svc_checker`: **Properties** → **Account** → untick **Password never expires**.

After this the flag still hadn't appeared, so I checked PowerShell for other enabled accounts with the flag set:

```powershell
PS C:\Windows\system32> Get-ADUser -Filter {PasswordNeverExpires -eq $true -and Enabled -eq $true}

DistinguishedName : CN=Administrator,CN=Users,DC=forela,DC=local
Enabled           : True
GivenName         :
Name              : Administrator
ObjectClass       : user
ObjectGUID        : 06ff120b-a7f7-498f-9280-9a987cb2629d
SamAccountName    : Administrator
SID               : S-1-5-21-2218304191-1582571568-1749508597-500
Surname           :
UserPrincipalName :
```

4. `Administrator` also has it set, so change that account's setting too:
   ![](../Pasted%20image%2020261104120339.png)

- Flag (after changing all 3 accounts):
  ![](../Pasted%20image%2020261104120654.png)

### Task 8: Which attribute needs to be modified to remediate the finding "Unprivileged users can add computer accounts to the domain"?

![](../Pasted%20image%2020261104120808.png)

- Answer: ms-DS-MachineAccountQuota

### Task 9: Reconfigure the domain to prevent unprivileged users from joining new computer accounts to Active Directory.

Despite being a low severity finding, this is a commonly abused attack vector.

**Option 1: GUI (Attribute Editor)**
1. Open **Active Directory Users and Computers** → **View** → enable **Advanced Features**.
2. Right-click **forela.local** → **Properties** → **Attribute Editor**.
3. Set `ms-DS-MachineAccountQuota` to `0`.
   ![](../Pasted%20image%2020261104130118.png)

**Option 2: PowerShell**

Query the current quota:

```powershell
Get-ADObject ((Get-ADDomain).distinguishedname) -Properties ms-DS-MachineAccountQuota
```

```
DistinguishedName         : DC=forela,DC=local
ms-DS-MachineAccountQuota : 10
Name                      : forela
ObjectClass               : domainDNS
ObjectGUID                : 5ba05303-9801-4954-ae91-38fd8e2a41bd
```

Set it to 0:

```powershell
Set-ADDomain -Identity (Get-ADDomain).distinguishedname -Replace @{"ms-DS-MachineAccountQuota"=0}
```

- Flag: `HTB{m4ch1n3_4cc0unt_qu0t4_z3r03d}`

### Task 10: What is the ServicePrincipalName associated with the finding "Privileged users with SPN defined"?

![](../Pasted%20image%2020261104124754.png)

- Answer: BackupSvc/DC01.forela.local

### Task 11: According to the "gMSA not in use" finding, which Operating System introduced the group Managed Service Account (gMSA) feature?

![](../Pasted%20image%2020261104125418.png)

- Answer: Windows Server 2016

### Task 12: How many SPNs are configured to support RC4 encryption?

![](../Pasted%20image%2020261104125552.png)

- Answer: 4

### Task 13: Mitigate Kerberoasting risks across the environment.

**1) Strip unnecessary SPNs from `Administrator`:**

```powershell
Set-ADUser -Identity Administrator -Remove @{servicePrincipalName="BackupSvc/DC01.forela.local"}
```

**2) Replace `svc_iis` by migrating its SPN to a new gMSA authorized for `WEBSERVER01$`:**

Remove the SPN from `svc_iis`:

```powershell
Set-ADUser -Identity svc_iis -Remove @{servicePrincipalName="HTTP/web.forela.local"}
```

Create a new gMSA with the same SPN. Setting `PrincipalsAllowedToRetrieveManagedPassword` to `WEBSERVER01$` makes that computer the only AD object able to retrieve the new service account's password:

```powershell
New-ADServiceAccount -Name "gmsa_iis" `
    -DNSHostName "web.forela.local" `
    -PrincipalsAllowedToRetrieveManagedPassword "WEBSERVER01$" `
    -ServicePrincipalNames "HTTP/web.forela.local"
```

**3) Restrict `svc_sql` to strong AES encryption types:**

```powershell
Set-ADUser -Identity "svc_sql" -Replace @{'msDS-SupportedEncryptionTypes' = 24}
```

> [!info] `msDS-SupportedEncryptionTypes` bitmask
> This attribute uses a bitmask to set which Kerberos encryption algorithms AD permits for an account. `24` is the standard value for AES-only:
> - `8`: AES-128 (AES128-CTS-HMAC-SHA1-96)
> - `16`: AES-256 (AES256-CTS-HMAC-SHA1-96)
> - `24` = 8 + 16: both AES-128 and AES-256, no RC4

- Flag:
  ![](../Pasted%20image%2020261104132140.png)

### Task 14: According to the finding "Users with Kerberos pre-authentication disabled", what is the name of the attack that this configuration leaves accounts vulnerable to?

![](../Pasted%20image%2020261104131950.png)

- Answer: ASREP-Roasting

### Task 15: Remediate the finding by removing the configuration from the account listed.

```powershell
Set-ADAccountControl -Identity svc_backup -DoesNotRequirePreAuth $false
```

- Flag:
  ![](../Pasted%20image%2020261104132452.png)

### Task 16: What is the DistinguishedName of the object with Unconstrained Delegation configured?

![](../Pasted%20image%2020261104132523.png)

- Answer: CN=WEBSERVER01,CN=Computers,DC=forela,DC=local

### Task 17: Reduce the delegation risk on the object by implementing Constrained Delegation restricted strictly to `MSSQLSvc/db.forela.local:1433`.

First, find the account linked to that SPN:

```powershell
Get-ADUser -Filter {ServicePrincipalNames -like "*MSSQL*" -and Enabled -eq $true}
```

```
DistinguishedName : CN=svc_sql,CN=Users,DC=forela,DC=local
Enabled           : True
GivenName         :
Name              : svc_sql
ObjectClass       : user
ObjectGUID        : 6cf3a8de-f511-44a5-99f9-92ea44f58207
SamAccountName    : svc_sql
SID               : S-1-5-21-2218304191-1582571568-1749508597-1107
Surname           :
UserPrincipalName :
```

Then remove the unconstrained delegation and add the constrained one:

```powershell
# First remove the existing delegation
Set-ADComputer -Identity "WEBSERVER01" -TrustedForDelegation $false
# Add the constrained delegation
Set-ADComputer -Identity "WEBSERVER01" -Add @{'msDS-AllowedToDelegateTo'="MSSQLSvc/db.forela.local:1433"}
```

- Flag:
  ![](../Pasted%20image%2020261104133740.png)

### Task 18: What is reported as the Identity of the AD Object associated with the Critical finding "Non-default principals with DC Sync rights on the domain"?

![](../Pasted%20image%2020261104133519.png)

- Answer: FORELA\Auditors

### Task 19: Sever the non-administrative DCSync attack path by revoking directory replication rights assigned to the object reported in the finding.

1. In **Active Directory Users and Computers** (Advanced Features on), right-click **forela.local** → **Properties** → **Security**.
2. Select **Auditors (FORELA\Auditors)** and untick **Replicating Directory Changes** and **Replicating Directory Changes All**.
   ![](../Pasted%20image%2020261104135602.png)

- Sherlock solved:
  ![](../Pasted%20image%2020261104135418.png)
