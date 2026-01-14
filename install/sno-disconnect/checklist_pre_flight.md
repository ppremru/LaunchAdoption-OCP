# Pre-Flight Go/No-Go Checklist

Created Date: January 14, 2026

Before booting the physical SNO hardware from the generated `agent.iso`, complete this checklist to ensure all air-gapped infrastructure dependencies are met. Failure to verify these items often results in installation hangs that are difficult to debug in the field.

---

## Pre-Boot Infrastructure Verification

| Category | Checkpoint | Status |
| --- | --- | --- |
| Network | Static IP for the SNO node is reserved and not in use by another host | [ ] |
| — | NTP server is reachable from the SNO node's network segment | [ ] |
| — | Gateway and Subnet Mask in `agent-config.yaml` match the local network | [ ] |
| DNS | `api.<cluster>.<domain>` resolves to the SNO Node IP | [ ] |
| — | `api-int.<cluster>.<domain>` resolves to the SNO Node IP | [ ] |
| — | `*.apps.<cluster>.<domain>` resolves to the SNO Node IP | [ ] |
| — | `<registry_fqdn>` resolves to the Disconnected Bastion IP | [ ] |
| Registry | The Quay registry is active and accessible via port `8443` | [ ] |
| — | The `pull-secret` in `install-config.yaml` contains the local registry auth | [ ] |
| — | The `additionalTrustBundle` includes the Registry/Root CA certificate | [ ] |

---

## Hardware Readiness Verification

| Component | Requirement | Status |
| --- | --- | --- |
| CPU | Minimum 8 physical cores (or vCPUs) available | [ ] |
| RAM | Minimum 16 GB to 32 GB of RAM available | [ ] |
| Disk | Primary installation disk is at least 120 GB | [ ] |
| — | Target disk is "clean" (no existing OCP/LVM metadata) | [ ] |
| BIOS/UEFI | Secure Boot is configured according to organizational policy | [ ] |
| — | Virtualization extensions (`VT-x`/`AMD-V`) are enabled in the BIOS | [ ] |

---

## Final Readiness Check

Run this final check from the **Disconnected Bastion** to ensure the registry is ready to serve the SNO node.

```bash
# Verify the OCP 4.16 release image is present in your local registry
oc adm release info <registry_fqdn>:8443/ocp4/openshift4-release:4.16.0-x86_64

```

---

## Architectural Justifications & Reference Notes

| Category | Technical Requirement Details | Documentation Source |
| --- | --- | --- |
| Time Sync | SNO is extremely sensitive to time drift. If the node clock differs from the registry/Bastion, the bootstrap certificates will fail to validate. | [SNO Preparing to Install](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_on_a_single_node/installing-sno-preparing-to-install-on-a-single-node) |
| DNS Resolution | The `api` and `api-int` endpoints must be resolvable before the installer starts, as the node will attempt to reach itself via these FQDNs. | [Installing on a single node](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_on_a_single_node/index) |
| ISO Verification | If the ISO fails to boot or hangs, verify the checksum of the `agent.iso` after transferring it to your boot media to rule out corruption. | [Agent-based Installer Guide](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_an_on-premise_cluster_with_the_agent-based_installer/index) |

---
