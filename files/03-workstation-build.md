# Phase 3 — Workstation Build

## Objective

Provision the employee workstation for the lab, verify its DNS
configuration against dc01, and join it to the `enterprise.local` domain.

## Step 1 — Provision win-client01

**Action:** Portal → `rg-hybrid-enterprise-lab` → Create → Virtual machine.

| Setting | Value | Purpose |
|---|---|---|
| Name | `win-client01` | identifies its role |
| Region | Central India | matches the virtual network |
| Image | Windows 11 Pro | standard employee-workstation OS |
| Size | Standard_B2as_v2 | balances capability against available Azure credit |
| Virtual network / Subnet | `vnet-enterprise-lab` / `subnet-client` | endpoint-tier subnet from Phase 1 |
| Private IP | Static, `10.10.2.4` | matches the architecture table |
| Public inbound ports | None | administered via Run Command |
| OS disk | Standard SSD (LRS), delete with VM | cost control |
| NIC network security group | None (relies on subnet-level `nsg-client`) | avoids duplicate rule sets |
| Auto-shutdown | Enabled, daily | protects remaining Azure credit |

**Evidence:** 📸 `images/03-01-winclient-create-review.png`

## Step 2 — Verify and correct DNS configuration

**Commands:**

```powershell
Get-DnsClientServerAddress -InterfaceAlias "Ethernet" -AddressFamily IPv4
```

**Purpose:** confirms which DNS server the network interface is actually
using. By default, a new Azure VM's NIC is set to "Inherited from virtual
network," which points at Azure's internal platform DNS
(`168.63.129.16`) rather than a domain controller. For a machine that will
join Active Directory, this must be explicitly overridden to point at
dc01, or the domain can never be located.

**Correction applied, in two places:**

- Azure NIC level: Portal → `win-client01` → Networking → NIC → DNS
  servers → Custom → `10.10.1.5`.
- OS level, to confirm the change took effect on the running interface:

```powershell
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses "10.10.1.5"
Get-DnsClientServerAddress -InterfaceAlias "Ethernet" -AddressFamily IPv4
```

**Result:** `ServerAddresses : {10.10.1.5}` confirmed on the Ethernet
interface.

**Final verification — actual name resolution, not just reachability:**

```powershell
Resolve-DnsName -Name enterprise.local
```

**Evidence:** 📸 `images/03-02-dns-server-check.png`

## Step 3 — Join win-client01 to the domain

**Command:**

```powershell
$cred = New-Object System.Management.Automation.PSCredential("ENTERPRISE\<dc01-local-admin-username>", (ConvertTo-SecureString "<dc01-local-admin-password>" -AsPlainText -Force))
Add-Computer -DomainName "enterprise.local" -Credential $cred -Restart -Force
```

**Purpose:** joins win-client01 to `enterprise.local`, authenticating as
the domain's built-in Administrator account. This account is not literally
named `Administrator` — Azure blocks that as a reserved VM username — it is
whichever local administrator account was defined when dc01 was created in
Phase 2, which became the domain's Administrator account upon promotion.
`-Restart` triggers the required reboot to complete the join.

**Note on output:** because `-Restart` reboots the machine mid-command, Run
Command's status report is sometimes lost in the restart, the same
behavior observed during dc01's promotion in Phase 2. The actual result
must be confirmed independently after the reboot, not assumed from the Run
Command output alone.

**Evidence:** 📸 `images/03-03-domain-join-result.png`

## Step 4 — Confirm the join

**Command (run after the reboot, ~3-5 min):**

```powershell
Get-ComputerInfo | Select-Object CsDomain, CsPartOfDomain
```

**Result:** `CsDomain: enterprise.local`, `CsPartOfDomain: True` — confirmed.

**Evidence:** 📸 `images/03-04-domain-confirmed.png`

## Outcome

win-client01 is provisioned, correctly configured to use dc01 for DNS, and
successfully joined to the `enterprise.local` domain.

**Next phase:** `04-kerberos-fundamentals.md`
