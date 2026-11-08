# Azure Active Directory & Kerberos Fundamentals Lab

A self-built Azure environment demonstrating core enterprise identity
infrastructure: network segmentation, a promoted Active Directory domain
controller, a domain-joined Windows client, and a real, captured Kerberos
authentication cycle — correlated against the matching Windows Security
events.

This project focuses on depth over breadth: four phases, each fully built,
documented, and backed by real command output and screenshots — rather than
a longer list of partially finished phases.

## What this project demonstrates

- **Azure infrastructure** — resource groups, virtual networks, role-based
  subnetting, and per-subnet Network Security Groups following a
  zero-trust, least-privilege network design
- **Active Directory** — promoting a Windows Server to a domain controller,
  AD DS and DNS integration, domain-joining a client machine
- **Kerberos authentication** — the real TGT/TGS ticket exchange, captured
  with `klist` and correlated against Windows Event IDs 4768 and 4769 on
  the domain controller
- **Operational troubleshooting** — diagnosing and resolving real issues
  encountered during the build (DNS misconfiguration, a stuck Run Command
  extension state, RDP permission errors), documented as they occurred

## Environment

| Machine | Role | Private IP | OS |
|---|---|---|---|
| dc01 | Domain Controller (AD DS, DNS, Kerberos KDC) | 10.10.1.5 | Windows Server 2022 |
| win-client01 | Domain-joined workstation | 10.10.2.4 | Windows 11 Pro |

Domain: `enterprise.local`

VNet: `10.10.0.0/16`, segmented into `subnet-dc`, `subnet-client`, and
`subnet-splunk` (reserved for a future extension — see below)

No machine in this environment was ever given a public inbound port for
longer than the single, scoped exception documented in Phase 4 — every
other administrative action was performed through Azure's Run Command
feature rather than direct network exposure.

## Repository layout

```
azure-ad-kerberos-lab/
├── README.md        <- this file
├── docs/            <- one file per phase, in build order
└── images/          <- screenshots, named to match their doc
```

## Phases

**01 — Azure Infrastructure**
Resource group, virtual network, and three role-segmented subnets, each
with its own NSG denying all inbound internet traffic.

**02 — Domain Controller Build**
Provisioned dc01 and promoted it to the first domain controller of a new
Active Directory forest (`enterprise.local`). Validated AD DS, DNS, and the
Kerberos KDC.

**03 — Workstation Build**
Provisioned win-client01, diagnosed and corrected a DNS misconfiguration
(the NIC defaulted to Azure's platform DNS instead of the domain
controller), and joined it to the domain.

**04 — Kerberos Fundamentals**
Created a standard domain user, captured a real TGT and service ticket
with `klist`, and correlated both against Event IDs 4768 and 4769 on the
domain controller's Security log.

## Possible extensions (not built)

The subnet layout reserves `subnet-splunk` for a natural next step: a
centralized Splunk instance ingesting these Windows Security events, plus
a controlled Kerberoasting simulation against the ticket mechanism
demonstrated in Phase 4. This was scoped out of the current build due to
Azure free-credit limits, not due to any technical blocker — the
architecture was deliberately left ready for it.

## Status

Complete for the stated scope (Phases 1-4). Screenshots and command output
for every step are in `docs/` and `images/`.
