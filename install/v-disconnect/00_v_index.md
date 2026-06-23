# Disconnected OpenShift Virtualization Deployment Index

Created Date: June 23, 2026
Status: Production Blueprint
Target Environment: Multi-Node Bare-Metal (High Availability)

---

## Why This Guide Exists

* **The Documentation:** Red Hat’s official guides serve as the comprehensive technical reference for the platform, organizing features by individual components.
* **This Guide:** This document acts as a streamlined, chronological checklist that pulls the documentation for the individual comonents together into a single workflow.

---

## Deployment Blueprint Matrix

The installation is segmented into five distinct operational phases. Each phase consumes the outputs of the previous step and must be fully validated before moving forward.

| File Reference | Operational Phase | Current Status | Primary Milestone / Output |
| --- | --- | --- | --- |
| `01_v_bastion_prep.md` | Staging the Connected Bastion | Environment Preparation | Validated CLI tools and a unified online/offline pull secret. |
| `02_v_mirroring_content.md` | Content Mirroring for the Air-Gap | Data Collection | Stateful payload bundle containing platform and operator layers. |
| `03_v_registry_setup.md` | Local Registry & Air-Gapped Ingestion | Registry Configuration | Populated internal OCI registry and cryptographic mirror tables. |
| `04_v_infrastructure_config.md` | Infrastructure & Media Compilation | Manifest Definition | Static host network maps and a bootable Agent installer ISO. |
| `05_v_virtualization_enable.md` | Virtualization & Day 2 Enablement | Post-Install Validation | Activated hypervisor engine and layer-2 guest bridging policies. |

---

## Operational Workflow Overview

### 1. Software Logistics Foundation (Steps 1–3)
Before a single piece of server hardware is unboxed or booted inside the air-gapped data center, the entire software logistics pipeline must be treated as a physical supply chain. In an isolated environment, software cannot be fetched on demand; it must be treated as physical inventory that is staged, quality-checked, and warehoused.

* **Quality Control at the Border (`01_bastion_prep.md`):** The connected bastion acts as our shipping dock. We download binaries and immediately execute cryptographic checksum verifications. This ensures we do not clear corrupted or tampered "inventory" to cross the air-gap perimeter.
* **Manifest Bundling & Transit (`02_mirroring_content.md`):** We execute a unified mirroring payload to pull the platform and operators down to physical transport media simultaneously. This locks their inter-operator version dependencies before they leave the network.
* **The Local Warehouse (`03_registry_setup.md`):** Once across the perimeter, the transport media is unladen, and the image layers are ingested directly into the internal OCI registry. This registry functions as our local warehouse, completely replacing public external endpoints as the sole root of trust for the environment.

### 2. Platform & Hypervisor Core Execution (Steps 4–5)
With the local warehouse fully stocked, the installation transitions from software logistics to physical cluster manufacturing.

* **Blueprint Definition (`04_infrastructure_config.md`):** Logical cluster pools and physical host identity maps—including MAC addresses and static IP routing tables—are defined in parallel layout manifests. 
* **Self-Contained Assembly (`05_virtualization_enable.md`):** The installer engine consumes these static definitions to compile a single, bootable installation ISO. Because our software logistics pipeline was perfectly executed in Steps 1–3, this ISO contains every cryptographic mirror table required to build the multi-node hypervisor cluster completely natively offline, with zero dependency on external PXE or DHCP networks.

---

## Core Master References

These authoritative Red Hat product streams form the baseline syntax reference for this entire deployment structure:

* **Platform Documentation:** [Red Hat OpenShift Container Platform 4.16 Documentation Portal](https://docs.redhat.com/en/documentation/openshift_container_platform/)
* **Mirroring Utility Engine:** [Mirroring Images via the oc-mirror Plugin Guide](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/mirroring_images_for_a_disconnected_installation_using_the_oc-mirror_plugin/)
* **Orchestrated Provisioning:** [Installing a Cluster via the Agent-Based Installer](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_a_cluster_using_the_agent-based_installer/)
* **Hypervisor Lifecycle:** [OpenShift Virtualization Core Installation and Administration](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/openshift_virtualization/installing-virt)
* **Node Networking Control:** [Advanced Interface Management with the Kubernetes NMState Operator](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/networking/index#virt-networking-with-kubernetes-nmstate)
