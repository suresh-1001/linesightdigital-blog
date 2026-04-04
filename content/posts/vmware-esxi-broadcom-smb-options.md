---
title: "VMware ESXi After Broadcom: What Bay Area SMBs Should Do Now"
date: 2026-04-03
draft: false
categories: ["Infrastructure", "Cloud"]
tags: ["VMware", "ESXi", "vSphere", "Broadcom", "Virtualization", "SMB", "Bay Area", "Azure", "Migration"]
description: "Broadcom's acquisition of VMware changed the licensing model in ways that hit SMBs hardest. Here's a clear-eyed look at your options — and how to think through the decision."
image: "/images/vmware-esxi-broadcom-smb-options.webp"
---

If you're running VMware ESXi in a small or mid-sized business environment, the last 18 months have been uncomfortable.

Broadcom acquired VMware in late 2023 and moved quickly to restructure licensing in ways that disproportionately affect smaller organizations. The free ESXi hypervisor was discontinued. Licensing moved to subscription-only bundles. Per-CPU pricing shifted to per-core. Support contracts that used to cost thousands now cost significantly more — or require a minimum commitment that doesn't make sense at SMB scale.

This isn't FUD. It's the reality that Bay Area SMBs are navigating right now.

This post gives you a clear framework for evaluating your options — without vendor bias.

## What Actually Changed

Before the Broadcom acquisition, VMware had multiple accessible paths for SMBs:

- **Free ESXi** (vSphere Hypervisor) — a fully functional hypervisor with no licensing cost, limited to standalone management
- **vSphere Essentials** — entry-level licensing for up to three hosts, reasonably priced for small IT environments
- **Per-CPU licensing** — predictable pricing if you knew your socket count

After the restructuring:

- Free ESXi was pulled. Existing free licenses stopped working after a grace period.
- vSphere Essentials was discontinued as a standalone product
- Licensing now bundles compute, storage, and networking components (VCF — VMware Cloud Foundation) that many SMBs simply don't need
- Minimum commitments and per-core pricing make small deployments significantly more expensive

For an SMB running 2–4 ESXi hosts with modest VM density, the cost increase has ranged from significant to untenable.

## Your Real Options

### Option 1: Stay on VMware — Evaluate the True Cost

This is the right choice if:

- Your VMware skill set is deeply embedded in your team
- You have complex workloads that depend on vSphere-specific features (vMotion, HA clusters, DRS)
- You've negotiated a reasonable Broadcom subscription contract

The honest math: if you're running ESXi on 2–4 hosts with a handful of VMs and your use case is straightforward, you're probably overpaying for capabilities you don't need. If you have a multi-site vSphere cluster with vCenter, DRS, and vSAN, the migration calculus is different.

Get a quote from Broadcom directly or through a reseller. Then compare it against the alternatives below.

### Option 2: Proxmox VE — The Most Common SMB Migration Target

Proxmox Virtual Environment is an open-source hypervisor built on KVM and LXC. It's been around since 2008, is actively developed, and has a large community and commercial support ecosystem.

**Why SMBs are migrating to Proxmox:**

- Free to use with optional paid support subscriptions ($119–$499/year per node)
- Familiar web-based management interface
- Supports live migration, clustering, and high availability
- Native ZFS storage integration
- VM import tools for migrating from ESXi

**What the migration actually involves:**

- Export VMs from ESXi as OVF/OVA
- Import into Proxmox (varying complexity by VM and guest OS)
- Reconfigure networking (Proxmox uses Linux bridges — different from vSwitch)
- Validate application behavior post-migration
- Update backup jobs, monitoring, and runbooks

For most SMB workloads — Windows Server VMs, Linux application servers, domain controllers — the migration is straightforward. For VMs with complex storage configurations or VMware Tools dependencies, it requires more planning.

We've done this migration multiple times. The process is well-understood.

### Option 3: Migrate Workloads to Azure or Cloud

This is the right path if:

- Your on-premises hardware is aging and due for refresh anyway
- Your workloads are stateless or relatively portable
- You need disaster recovery capabilities you currently lack
- Your team is already working toward a cloud-forward model

**Azure Migration Center** provides tools for assessing and migrating VMware workloads directly to Azure VMs. **Azure Migrate** can assess your ESXi environment and produce a cost model before you commit to anything.

The economics depend heavily on your workload profile. Lift-and-shift of resource-intensive VMs to Azure can be expensive at steady state. The right approach is often a hybrid: move some workloads to cloud, consolidate others onto a smaller on-premises footprint, and decommission the rest.

We design these migrations regularly for Bay Area SMBs and can model the cost comparison honestly.

### Option 4: Microsoft Hyper-V

Often overlooked, Hyper-V is included with Windows Server licenses you may already own. If your environment is predominantly Windows-based, it's worth evaluating.

**Where Hyper-V makes sense:**

- You have Windows Server Datacenter licenses (which include unlimited Windows VM rights)
- Your team is already comfortable in the Microsoft ecosystem
- You're managing infrastructure through Intune or System Center

**Where it doesn't:**

- Linux VM workloads (Hyper-V Linux support is functional but less mature than KVM)
- Complex networking requirements
- Any preference for an open-source management layer

## How to Decide

The right answer depends on four factors:

1. **Hardware lifecycle** — If your ESXi hosts are 3–4 years old and facing refresh, that changes the math significantly. A refresh is an opportunity to reconsider the platform.

2. **Workload complexity** — Simple Windows/Linux VMs migrate easily. Complex clustered applications with shared storage require careful planning regardless of destination.

3. **Skill set** — If your internal team (or your MSP) has deep VMware expertise, that's a real asset. Migrating away means retraining or bringing in external help during the transition.

4. **Business trajectory** — Are you growing headcount and complexity? Moving toward cloud? Considering acquisition? Your IT platform should support the business direction, not just the current state.

## What a VMware Assessment Looks Like

When we engage with an SMB that's wrestling with this decision, we start with a structured assessment:

- Inventory of hosts, VMs, and resource utilization
- Current licensing cost vs. projected Broadcom renewal cost
- Workload categorization (cloud-ready, migrate-to-Proxmox, stay on VMware)
- Hardware lifecycle timeline
- Comparison of total cost of ownership across 3 years for each path

Most organizations we work with end up with a hybrid recommendation: migrate some workloads to Azure, consolidate the remainder onto Proxmox or Hyper-V, and eliminate the VMware licensing cost entirely.

## What LineSight Digital Offers

With over 20 years of VMware ESXi and vSphere experience, we help Bay Area SMBs navigate this decision without vendor pressure.

Our VMware advisory and migration services include:

- ESXi environment assessment and Broadcom cost modeling
- Migration planning for Proxmox VE, Azure, or Hyper-V
- VM export, import, and validation
- Networking reconfiguration and documentation
- Post-migration monitoring and support
- Veeam backup reconfiguration for new platform

We've managed ESXi environments ranging from single-host home labs to multi-site vSphere clusters with vSAN. We know what migrating away actually costs in time and risk — and we'll give you an honest assessment.

**[Schedule a free IT assessment](https://linesightdigital.com/#contact)** — we'll review your current VMware environment and help you figure out the right path forward.

### Related Posts

- [5 Signs Your Bay Area Startup Needs a Fractional IT Director](https://blog.linesightdigital.com/posts/5-signs-startup-needs-fractional-it-director/)
- [Backup and Disaster Recovery for Bay Area Small Businesses](https://blog.linesightdigital.com/posts/backup-disaster-recovery-small-business/)
