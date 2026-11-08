# Phase 1 — Azure Infrastructure

## Objective

Establish the foundational Azure infrastructure for the lab: a dedicated
resource group, an isolated virtual network, and role-based subnetting that
mirrors how a real enterprise network segments identity infrastructure,
endpoints, and monitoring systems.

## Step 1 — Create the resource group

**Action:** Portal → Create a resource → Resource group.

| Setting | Value |
|---|---|
| Name | `rg-hybrid-enterprise-lab` |
| Region | Central India |

**Purpose:** the resource group serves as the logical container for every
resource in the lab, allowing the entire environment to be managed,
monitored, and (if needed) torn down as a single unit.

**Evidence:**  ![resource group](images/01-01-resource-group-created.png) — the resource
group's Overview blade immediately after creation.

## Step 2 — Create the virtual network and subnets

**Action:** Portal → Create a resource → Virtual network.

| Setting | Value | Purpose |
|---|---|---|
| Name | `vnet-enterprise-lab` | |
| Address space | `10.10.0.0/16` | sized to accommodate all planned subnets |
| Region | Central India | matches the resource group |

On the **IP Addresses** tab, define three subnets:

| Subnet name | Address range | Hosts |
|---|---|---|
| `subnet-dc` | 10.10.1.0/24 | domain controller |
| `subnet-client` | 10.10.2.0/24 | workstation |
| `subnet-splunk` | 10.10.3.0/24 | Splunk server |

**Purpose:** segmenting the network by role — identity infrastructure,
endpoint, and monitoring — rather than placing all machines on a single
flat subnet avoids a single point of failure if one subnet is compromised,
and lets network security policy be scoped per tier in the following step.
This follows the zero-trust and least-privilege principles used in real
enterprise network design, rather than granting every machine the same
level of network access by default.

**Evidence:** 📸 `images/01-02-vnet-subnets.png` — the virtual network's
Subnets blade showing all three ranges.

## Step 3 — Create and associate a Network Security Group per subnet

**Action:** Portal → Create a resource → Network security group, repeated
for each subnet:

| NSG name | Associated subnet |
|---|---|
| `nsg-dc` | `subnet-dc` |
| `nsg-client` | `subnet-client` |
| `nsg-splunk` | `subnet-splunk` |

For each NSG: NSG → Subnets → Associate → select its corresponding subnet.

**Purpose:** the default NSG ruleset denies all inbound traffic from the
public internet while permitting traffic within the virtual network. No
inbound rule is added for RDP, SSH, or any management port — every machine
in this lab is administered through Azure's Run Command feature rather than
direct network exposure. This establishes a baseline security posture in
which no host is ever reachable from the public internet, which is stated
explicitly here because it is a verifiable architectural decision, not an
assumption.

**Evidence:** 📸 `images/01-03-nsg-associations.png` — the resource group's
resource list showing all three NSGs, or individual screenshots of each
NSG's Subnets blade confirming its association.

## Outcome

A resource group containing one virtual network, three role-segmented
subnets, and an NSG attached to each, with no inbound internet access
permitted anywhere in the environment.

**Next phase:** `02-domain-controller-build.md`
