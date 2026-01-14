---

# Single Node OpenShift (SNO) Disconnected Deployment Guide

**Created Date**: January 14, 2026

**Target Version**: OpenShift Container Platform 4.16

This documentation provides a hardened, end-to-end workflow for deploying a Single Node OpenShift (SNO) cluster in a disconnected environment. We utilize the **Agent-based Installer**, which embeds configurations directly into a bootable ISO, eliminating the need for external bootstrap infrastructure during the final boot.

---

## Working Environment Definitions

| System Component | Description & Technical Role | Network Placement |
| --- | --- | --- |
| Connected Bastion | RHEL 8/9 host used to mirror platform and operator data from Red Hat registries. | Internet Facing |
| Disconnected Bastion | RHEL 8/9 host used to build the Agent ISO and host the local mirror registry (Quay). | Air-Gapped |
| Target SNO Node | Physical server or VM where the OpenShift 4.16 cluster will be deployed. | Air-Gapped |

---

## Infrastructure & Network Requirements

Before beginning the implementation, ensure the following prerequisites are met within the air-gapped environment:

| Requirement | Description | Specifics |
| --- | --- | --- |
| Static IPs | Dedicated IP addresses for core infrastructure components. | One for Bastion |
| — | — | One for SNO Node |
| DNS Records | Resolvable records pointing to the single node IP. *.apps must point to SNO IP. | api, api-int, *.apps |
| — | — | registry-fqdn |
| NTP | Mandatory local time synchronization source. | Local NTP Server IP |
| Admin Credentials | Red Hat Portal ID and SSH Key Pair for node access. | Pull Secret, Public SSH Key |

---

## Pre-Flight Resource Validation

Failure to meet these minimums will cause the SNO installation or Day 2 operator deployments to fail. Ensure RHCOS version matches OCP 4.16.

| Category | Technical Requirement Justification | Documentation Source |
| --- | --- | --- |
| SNO Node Storage | A minimum of 120 GB is required for the base RHCOS installation. | [Installing on a single node](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_on_a_single_node/index) |
| Bastion Storage | 200 GB minimum accounts for platform images and common operators. | [Mirroring images for disconnected installation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/disconnected_installation_mirroring/index) |
| Registry Persistence | The Disconnected Bastion must house the Quay database and all layers permanently. | [Creating a mirror registry](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/disconnected_installation_mirroring/installing-mirroring-creating-mirror-registry) |
| SNO Compute | Minimum requirements are 8 vCPUs and 16-32 GB of RAM. | [SNO Preparing to Install](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_on_a_single_node/installing-sno-preparing-to-install-on-a-single-node) |
| Installer Specs | The installer host requires space for the binary and a ~1 GB bootable ISO. | [Agent-based Installer Guide](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_an_on-premise_cluster_with_the_agent-based_installer/index) |

---

## The Sneakernet Workflow

In a disconnected environment, the "Sneakernet" process is the manual method of bridging the air-gap using physical media (such as external SSDs).

| Phase | Action | Requirement |
| --- | --- | --- |
| Collection | Mirroring platform images and operators from Red Hat to local media. | Connected Bastion + Physical Media |
| Transition | Physically moving the media through security checkpoints to the air-gapped zone. | Secure Chain of Custody |
| Ingestion | Uploading mirrored content from media into the local disconnected registry. | Disconnected Bastion + Local Registry |

---

## Implementation Roadmap

| Phase | Objective |
| --- | --- |
| **Day 0: Preparation** | [Establish the connected staging environment and gather binaries](https://www.google.com/search?q=./01_bastion_prep.md) |
| — | [Declaratively mirror OCP images and operators for physical transfer](https://www.google.com/search?q=./02_mirroring_content.md) |
| — | [Deploy and harden a local Quay registry for air-gapped ingestion](https://www.google.com/search?q=./03_registry_setup.md) |
| **Day 1: Installation** | [Define the SNO network and cluster logic via YAML manifests](https://www.google.com/search?q=./04_sno_configurations.md) |
| — | [Build the self-contained agent.iso and initiate the hardware boot](https://www.google.com/search?q=./05_iso_generation.md) |
| **Day 2: Hardening** | [Resolve common air-gap and certificate-related deployment faults](https://www.google.com/search?q=./trouble.md) |
| — | [Final verification of the environment prior to ISO execution](https://www.google.com/search?q=./checklist_pre_flight.md) |
| — | [Validating cluster health, storage, and supply chain integrity](https://www.google.com/search?q=./checklist_post_install.md) |
| — | [Implementing local storage (LVMS) and log aggregation](https://www.google.com/search?q=./day2.md) |

---

## Appendix: Methodology & Scope

While official Red Hat documentation provides foundational technical references for individual components, this guide serves as an architectural blueprint specifically synthesized for **high-security, disconnected environments.**

### The "Secret Sauce"

* **Synthesis of Fragmentation**: This guide eliminates the need to cross-reference multiple manuals by providing a single, linear assembly line for deployment.
* **Agent-Based Architecture**: Utilizing the Agent-based Installer creates a "Cluster in a Box," reducing external infrastructure dependencies.
* **Hardened Security by Default**: This methodology prioritizes established chain-of-trust protocols using Internal CAs and explicit certificate injection into the `additionalTrustBundle`.
* **Pre-Flight Rigor**: Explicit "Go/No-Go" gates verify DNS, NTP, and Registry availability before physical provisioning begins.

---
