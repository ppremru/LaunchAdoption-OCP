# Disconnected SNO v4.16 Installation Index

Created Date: January 14, 2026

Target Version: OpenShift Container Platform 4.16

This documentation provides a step-by-step GUIDE to deploying a Single Node OpenShift (SNO) cluster in a disconnected (air-gapped) environment using the Agent-based Installer.

## Installation Steps

| # | Step Description | 
| --- | --- |
| 1. | [Setting up the Connected Bastion and acquiring the Pull Secret.](./01_bastion_prep.md) |
| 2. | [Performing the initial mirror to physical media.](/02_mirroring_content.md) |
| 3. | [Setting up the Disconnected Bastion (Local Host) and the Mirror Registry.](./03_registry_setup.md) |
| 4. | [Crafting the YAML files for the specific hardware.](./04_sno_configurations.md) |
| 5. | [Final ISO creation and cluster verification.](./05_iso_generation.md) |

Extras:
* [Day 2 Notes](./day2.md)
* [Trouble Shooting Notes](./trouble.md)

## Key Concepts

| Concept | Description | Links |
| --- | --- | --- |
| Single Node OpenShift (SNO) | Combines control plane and worker roles onto one host. Optimized for edge and low-resource environments. Note: Hardware requirements (CPU, RAM, and Disk) must be reviewed prior to installation to ensure the host can support both the control plane and your intended workloads. | <ul><li>[Minimum resource requirements for Single Node OpenShift](https://www.google.com/search?q=https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_on_a_single_node/installing-sno-preparing-to-install-on-a-single-node%23minimum-resource-requirements-for-sno_installing-sno-preparing-to-install-on-a-single-node)</li><li>[Official SNO Overview](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_on_a_single_node/index)</li></ul> |
| oc-mirror Plugin v2 | The modern tool (v2) used to declaratively mirror images and operators to a local registry. | <ul><li>[oc-mirror Documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/disconnected_installation_mirroring/installing-mirroring-disconnected)</li></ul> |
| Agent-based Installer | Generates a bootable ISO containing all configurations, removing the need for an external bootstrap VM. | <ul><li>[Agent-based Installer Guide](https://www.google.com/search?q=https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_an_on-premise_cluster_with_the_agent-based_installer/index)</li></ul> |

## Working Environment Definitions

| System Component | Description & Technical Role | Network Placement |
| --- | --- | --- |
| Connected Bastion | RHEL 8/9 host used to mirror platform and operator data from Red Hat registries. | Internet Facing |
| Disconnected Bastion | RHEL 8/9 host used to build the Agent ISO and host the local mirror registry (Quay). | Air-Gapped |
| Target SNO Node | Physical server or VM where the OpenShift 4.16 cluster will be deployed. | Air-Gapped |

## Infrastructure & Network Requirements

| Requirement | Description | Specifics |
| --- | --- | --- |
| Static IPs | Dedicated IP addresses for core infrastructure components. | <ul><li>One for Bastion</li><li>One for SNO Node</li></ul> |
| DNS Records | Resolvable records for api, api-int, and wildcard apps pointing to the SNO node. Note: For SNO, all cluster records point to the same single node IP. | <ul><li>api</li><li>api-int</li><li>*.apps</li><li>registry-fqdn</li></ul> |
| NTP | Mandatory local time synchronization source. | <ul><li>Local NTP Server IP</li></ul> |
| Admin Credentials | Red Hat Customer Portal ID and SSH Key Pair for node access. | <ul><li>Pull Secret</li><li>Public SSH Key</li></ul> |

## Pre-Flight Resource Validation

| Category | Technical Requirement Justification | Documentation Source |
| --- | --- | --- |
| SNO Node Storage | A minimum of 120 GB of storage is required for the base RHCOS installation on a single node. | [Installing on a single node](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_on_a_single_node/index) |
| Bastion Storage | A 200 GB minimum recommendation accounts for platform images and common operators (e.g., LVM Storage, Logging). | [Mirroring images for disconnected installation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/disconnected_installation_mirroring/index) |
| Registry Persistence | The Disconnected Bastion must house the Quay registry database and all image layers permanently. | [Creating a mirror registry](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/disconnected_installation_mirroring/installing-mirroring-creating-mirror-registry) |
| SNO Compute | Minimum requirements for SNO are 8 vCPUs and 16-32 GB of RAM to support combined control plane and worker services. | [SNO Preparing to Install](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_on_a_single_node/installing-sno-preparing-to-install-on-a-single-node) |
| Installer Specs | The installer host requires enough space to run the openshift-install binary and generate a ~1 GB bootable ISO. | [Agent-based Installer Guide](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_an_on-premise_cluster_with_the_agent-based_installer/index) |

## The Sneakernet Workflow

In a disconnected environment, the "Sneakernet" process is the manual method of bridging the air-gap between the internet-facing network and the isolated environment using physical media (such as high-speed USB-C drives or external SSDs).

| Step Phase | Action | Requirement |
| --- | --- | --- |
| 1. Collection | Mirroring platform images and operator catalogs from Red Hat to local media. | Connected Bastion + Physical Media |
| 2. Transition | Physically moving the media through security checkpoints or data diodes to the air-gapped zone. | Secure Chain of Custody |
| 3. Ingestion | Uploading the mirrored content from the physical media into the local disconnected registry. | Disconnected Bastion + Local Registry |

### What to Expect from the Sneakernet Process

| Category | Technical Reality & Advice |
| --- | --- |
| Massive Data Footprint | A standard OpenShift 4.16 platform mirror plus essential operators (LVM, Logging) typically ranges between **100GB and 150GB**. Ensure your physical media is formatted (XFS or EXT4 recommended) and has at least 200GB of free space. |
| Transfer Speed Bottlenecks | The time required to "write" the data to media and then "read" it into the disconnected registry is often the longest phase of the installation. Use USB 3.1+ or NVMe-based external drives to minimize downtime. |
| Integrity Verification | Large data transfers are susceptible to silent corruption during the move. It is highly recommended to run a checksum (SHA-256) on the connected side and verify it once the data reaches the disconnected side before starting the ingestion. |

---
