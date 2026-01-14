# Step 5: ISO Generation & Verification (Disconnected)

Created Date: January 14, 2026

Status: Deployment Phase

This final document guides the administrator through the generation of the bootable Agent ISO and the subsequent cluster verification process. This phase completes the installation of the Single Node OpenShift (SNO) cluster in the air-gapped environment.

## ISO Generation Overview

| Task | Description |
| --- | --- |
| **ISO Creation** | Using the `openshift-install` binary to consume the YAML configurations and produce a bootable image. |
| **Node Boot** | Booting the physical hardware or VM from the generated ISO to initiate the automated installation. |
| **Cluster Verification** | Using the OpenShift CLI (`oc`) to monitor the installation progress and verify the health of the final node. |

### ISO Generation Reference Script

> **Note**: The following commands are a **functional guide**. Ensure your `install-config.yaml` and `agent-config.yaml` are in the same directory before execution.

```bash
# 1. Create the Agent ISO
# This command generates an 'agent.iso' in the specified directory
openshift-install agent create image --dir ./sno-config

# 2. Monitor the Installation
# Once the node has booted from the ISO, monitor progress from the Bastion
export KUBECONFIG=./sno-config/auth/kubeconfig
watch "oc get nodes; oc get clusteroperators"

# 3. Wait for Install Completion
# This command provides a real-time status update of the install process
openshift-install agent wait-for install-complete --dir ./sno-config

```

---

## Technical Process & Justifications

| Category | Justification |
| --- | --- |
| **1. Agent Image Command** | The `agent create image` command specifically instructs the installer to build a bootable ISO that includes the CoreOS image and your static network configurations. |
| **2. Kubeconfig Export** | Exporting the `KUBECONFIG` variable allows the `oc` tool to communicate with the new cluster's API as soon as it becomes available during the boot process. |
| **3. Cluster Operators** | Monitoring `clusteroperators` is the best way to verify that all internal services (networking, authentication, storage) have reached a "Stable" state. |

---

## Architectural Justifications & Reference Notes

| Category | Technical Requirement Details | Documentation Source |
| --- | --- | --- |
| **Boot Media** | For physical hardware, the `agent.iso` can be written to a USB drive or mapped via the BMC (iDRAC/iLO) virtual media feature. | [Installing on a single node](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_on_a_single_node/index) |
| **Post-Install Cleanup** | After a successful install, the temporary `sno-config` directory contains sensitive files like the `kubeadmin` password. Secure or delete these files according to your local security policy. | [Agent-based Installer Guide](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_an_on-premise_cluster_with_the_agent-based_installer/index) |
| **Final Registry Check** | Once the node is "Ready," verify that the SNO node is successfully pulling images from the Disconnected Bastion registry to confirm the air-gapped supply chain is functional. | [oc-mirror Documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/disconnected_installation_mirroring/installing-mirroring-disconnected) |

---
