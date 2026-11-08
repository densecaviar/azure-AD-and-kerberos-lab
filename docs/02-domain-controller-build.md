# Phase 2 — Domain Controller Build

## Objective

Provision the domain controller for the lab and promote it to the first
domain controller of a new Active Directory forest, establishing identity,
DNS, and Kerberos authentication services for the environment.

## Step 1 — Provision dc01

**Action:** created via Portal → Create a resource → Virtual machine.

| Setting | Value | Purpose |
|---|---|---|
| Name | `dc01` | identifies the machine's role |
| Region | Central India | matches the virtual network |
| Image | Windows Server 2022 Datacenter | required for the AD DS and DNS server roles |
| Size | Standard_B2as_v2 | balances capability against available Azure credit |
| Virtual network / Subnet | `vnet-enterprise-lab` / `subnet-dc` | identity-tier subnet from Phase 1 |
| Private IP | Static, `10.10.1.5` | matches the architecture table |
| Public inbound ports | None | administered via Run Command, consistent with the no-internet-exposure posture from Phase 1 |
| OS disk | Standard SSD (LRS), delete with VM | cost control |
| NIC network security group | None (relies on subnet-level `nsg-dc`) | avoids duplicate, overlapping rule sets |
| Auto-shutdown | Enabled, daily | protects remaining Azure credit |

**Evidence:** 📸 `images/02-01-dc01-create-review.png` — the Review + Create
summary prior to deployment.

## Step 2 — Install the required Windows Server roles

**Command (via Run command → RunPowerShellScript):**

```powershell
Install-WindowsFeature AD-Domain-Services, DNS -IncludeManagementTools
```

**Purpose:** installs the Active Directory Domain Services and DNS Server
roles, neither of which exists on a default Windows Server install. This is
a prerequisite for promotion.

**Result:** succeeded.

## Step 3 — Promote dc01 to a domain controller

**Command:**

```powershell
Install-ADDSForest -DomainName "enterprise.local" -DomainNetbiosName "ENTERPRISE" -InstallDns:$true -SafeModeAdministratorPassword (ConvertTo-SecureString "Lab@Dc01Secure#2026" -AsPlainText -Force) -Force:$true -NoRebootOnCompletion:$true
```

**Purpose:** creates a new Active Directory forest rooted at
`enterprise.local`, promotes this machine to its first domain controller,
and installs DNS as an integrated part of the same operation.
`-SafeModeAdministratorPassword` sets the Directory Services Restore Mode
recovery password (a requirement of the cmdlet, unrelated to normal login).
`-NoRebootOnCompletion:$true` lets the command finish and report success
cleanly; the reboot is then triggered manually as a separate step, avoiding
a known issue where an automatic mid-command reboot kills Run Command's
status reporting before it can confirm completion.

**Result:** succeeded, with three informational warnings that do not
indicate failure (NTLM compatibility setting, a note on static IP best
practice, and a DNS delegation note relevant only when integrating with an
existing external DNS hierarchy — not applicable here).

**Action:** VM manually restarted via Portal → `dc01` → **Restart**.

## Step 4 — Validate domain controller health

**Commands:**

```powershell
Get-Service NTDS, DNS, Kdc | Select-Object Name, Status
Get-ADDomain | Select-Object DNSRoot, NetBIOSName, DomainMode
dcdiag /v > C:\dcdiag-output.txt
Get-Content C:\dcdiag-output.txt | Select-String "failed"
Resolve-DnsName -Name _ldap._tcp.dc._msdcs.enterprise.local -Type SRV
```

**Purpose of each:**
- `Get-Service` — confirms the AD database engine, DNS, and Kerberos KDC
  are all running.
- `Get-ADDomain` — confirms the domain is live and queryable.
- `dcdiag /v` piped through `Select-String "failed"` — runs the full
  domain controller diagnostic suite and surfaces only failing tests,
  since the full report exceeds the Run Command output size limit.
- `Resolve-DnsName` — confirms the SRV record that lets other machines
  locate this domain controller has registered correctly.

**Result — `dcdiag`:** one failing test, `DFSREvent` (DFS Replication
service, used to sync SYSVOL between multiple domain controllers). This is
expected and non-blocking on a single-DC forest, since there is no second
DC to replicate to; it does not affect AD DS, DNS, or Kerberos.

**Result — SRV record resolution:**
```
Name                                     Type   TTL   Section    NameTarget                     Priority Weight Port
----                                     ----   ---   -------    ----------                     -------- ------ ----
_ldap._tcp.dc._msdcs.enterprise.local    SRV    600   Answer     dc01.enterprise.local          0        100    389

Name       : dc01.enterprise.local
QueryType  : A
TTL        : 3600
Section    : Additional
IP4Address : 10.10.1.5
```

The SRV record resolves correctly to dc01's static IP, confirming clients
can locate this domain controller.

**Evidence:**
📸 `images/02-02-adds-forest-result.png` — Run Command output confirming
the promotion succeeded.
📸 `images/02-03-validation-output.png` — `Get-Service` and `Get-ADDomain`
output.
📸 `images/02-04-srv-record-resolved.png` — the SRV record output above.

## Outcome

A validated, fully promoted domain controller for `enterprise.local`,
providing Active Directory, DNS, and Kerberos authentication services to
the environment. The one non-passing diagnostic test (DFSREvent) is
expected and documented, not an unresolved issue.

**Next phase:** `03-workstation-build.md`
