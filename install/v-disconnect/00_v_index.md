# Disconnected OpenShift Virtualization Deployment Guide

**Created Date**: June 23, 2026
**Target Version**: OpenShift Container Platform 4.16

This documentation provides a chronological deployment checklist for building a highly available, multi-node OpenShift Container Platform cluster with active OpenShift Virtualization natively offline. We utilize the **Agent-based Installer** to embed static network configurations and local registry mirror definitions directly into a self-contained bootable ISO, eliminating external DHCP, PXE, or bootstrap infrastructure requirements during hardware provisioning.

---

## Working Environment Definitions

| System Component | Description & Technical Role | Network Placement |
| --- | --- | --- |
| Connected Bastion | RHEL 8/9 host used to securely download client binaries and mirror platform/operator images via `oc-mirror`. | Internet Facing |
| Local Mirror Registry | Internal Quay enterprise registry deployed on port 8443 acting as the local software warehouse repository. | Air-Gapped |
| Target Cluster Nodes | Three (3) physical control plane masters and multiple physical compute workers dedicated to hypervisor workloads. | Air-Gapped |

---

## Infrastructure & Network Requirements

Before beginning the hardware deployment phase, ensure the following core networking gates are active within the air-gapped data center:

| Requirement | Description | Specifics |
| --- | --- | --- |
| Static IP Pools | Isolated IPs assigned to master nodes, worker nodes, and logical VIP layers. | Individual Node IPs + API and Ingress VIPs |
| Offline DNS Records | Internal resolvable records mapping cluster routing and local warehouse lookups. | `api`, `api-int`, `*.apps`, and your internal `<registry_fqdn>` |
| Local NTP Source | Mandatory time synchronization engine to prevent database consensus failure. | Local Stratum-1 NTP Server IP |
| Switch Port Fabrics | Worker node secondary ports (e.g., `eth1`) bound to virtual networks. | Physical Switch Ports configured as L2 VLAN Trunks |

---

## Pre-Flight Resource Validation

Failure to satisfy these baseline resource metrics will block the initialization sequence or cause down-stream hypervisor deployment failures.

| Category | Technical Requirement Justification | Documentation Source |
| --- | --- | --- |
| Node OS Storage | Each bare-metal server requires a dedicated boot drive (**120 GB–200 GB+** SSD/NVMe RAID-1) isolated from VM data disks. | [Bare-Metal Cluster Infrastructure Requirements](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_on_bare_metal/index) |
| Bastion Storage | **500 GB+** available on the active mirror partition to prevent mid-stream workspace exhaustion during catalog syncs. | [Mirroring Images for Disconnected Environments](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/mirroring_images_for_a_disconnected_installation_using_the_oc-mirror_plugin/) |
| VM Data Storage | Target virtual machine drives present inside compute worker nodes must be left completely **raw and unpartitioned**. | [OpenShift Virtualization Storage Planning](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/openshift_virtualization/installing-virt) |
| Compute Hardware | Physical worker servers must have bare-metal hypervisor extensions enabled natively within the system BIOS/UEFI. | [OpenShift Virtualization Core Installation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/openshift_virtualization/installing-virt) |

---

## The Sneakernet Workflow

In a completely disconnected environment, the "Sneakernet" process serves as our secure physical bridge to ferry data payloads across the secure perimeter.

| Phase | Action | Requirement |
| --- | --- | --- |
| Collection | Mirroring OCP platform channels and virtualization operator catalogs using `oc-mirror --v2` to physical media. | Connected Bastion + Removable Storage |
| Transition | Physically routing the encrypted storage media through strict corporate security checkpoints. | Verified Chain of Custody |
| Ingestion | Uploading the structured image layers directly into the local enterprise Quay registry over port 8443. | Offline Registry Bastion + Local Storage Media |

---

## Implementation Roadmap

| Phase | Objective |
| --- | --- |
| **Day 0: Preparation** | [Stage the connected environment and retrieve verified orchestrator tools](./01_v_bastion_prep.md) |
| — | [Declaratively mirror platform and virtualization operator images to local media](./02_v_mirroring_content.md) |
| — | [Deploy the internal warehouse registry and fuse localized trust tokens](./03_v_registry_setup.md) |
| **Day 1: Installation** | [Map physical host identities, static routing rules, and logical cluster parameters](./04_v_infrastructure_config.md) |
| — | [Compile the self-contained Agent boot media and provision bare-metal nodes](./04_v_infrastructure_config.md) |
| **Day 2: Hardening** | [Subscribe to hypervisor components and isolate high-throughput live migration traffic](./05_v_virtualization_enable.md) |
| — | [Plumb layer-2 guest network bridges and expose them to local namespaces via Multus](./05_v_virtualization_enable.md) |

---

## Appendix: Methodology & Scope

Official Red Hat product guides serve as the comprehensive technical reference for individual platform elements. This implementation blueprint functions as a production flight checklist specifically synthesized to streamline multi-node virtualization deployments within high-security network zones.

### The "Secret Sauce"

* **Linear Assembly Line**: Eliminates disjointed context-switching by consolidation of mirroring, configuration, networking, and hypervisor setup into one timeline.
* **Infrastructure Independence**: Leveraging the Agent-Based Installer bypasses the need to maintain external, high-maintenance DHCP or PXE server configurations inside isolated zones.
* **Hardened Trust Enforced**: Enforces end-to-end cryptographic integrity by matching local container runtime authentication maps to the cluster's internal trust anchors.
* **Go/No-Go Gates**: Restricts actual deployment actions until underlying physical resources, time servers, and local registry ports are completely validated.
