# Troubleshooting & FAQ (Disconnected)

Created Date: January 14, 2026

Status: Operational Support

This document provides a technical baseline for identifying and resolving the most common issues encountered during a disconnected SNO 4.16 installation. Because the environment is air-gapped, traditional "just Google it" methods are unavailable; this guide serves as your local reference.

## Common Installation Faults

| Category | Observed Issue | Potential Root Cause & Resolution |
| --- | --- | --- |
| **Pull Failures** | Pods remain in `ImagePullBackOff` or `ErrImagePull`. | **Cause:** The node cannot reach the registry or lacks authentication. <br>

<br> **Resolution:** Verify the merged `pull-secret` includes the credentials for your local registry. Check firewall port 8443 on the Bastion. |
| **Trust Errors** | Errors like `x509: certificate signed by unknown authority`. | **Cause:** The SNO node does not trust the registry's CA. <br>

<br> **Resolution:** Ensure the Registry CA was added to the `additionalTrustBundle` in `install-config.yaml`. |
| **DNS Failures** | The install stalls at "Waiting for API." | **Cause:** Internal DNS is not resolving `api.<cluster>.<domain>`. <br>

<br> **Resolution:** Verify your local DNS records match the static IP defined in `agent-config.yaml`. |
| **Time Drift** | Etcd operators remain in a `Degraded` state. | **Cause:** The SNO node clock has drifted from the local source. <br>

<br> **Resolution:** Manually sync the hardware clock and verify the NTP server is reachable from the SNO node. |

---

## Technical Diagnostics Script

> **Note**: These commands are intended to be run from the **Disconnected Bastion** once the SNO node has partially booted and is reachable via the network.

```bash
# 1. Check Node Connectivity & SSH Access
# If the ISO boot works, you should be able to SSH into the node
ssh core@<node_ip>

# 2. Inspect the Installer Logs (On the SNO Node)
# Run these while SSH'd into the core user on the SNO node
journalctl -b -f -u agent-installer.service

# 3. Verify Local Registry Accessibility (On the SNO Node)
# Test if the node can actually talk to the registry API
curl -k https://<registry_fqdn>:8443/v2/_catalog

# 4. Monitor Operator Degradation (From Bastion)
# Identify which specific system component is failing
oc get clusteroperators | grep -v "True.*False.*False"

```

---

## Frequently Asked Questions (FAQ)

| Question | Technical Answer |
| --- | --- |
| **Why is my ISO so small?** | The `agent.iso` is a "Next Generation" installer. It does not contain the full OCP payload; it contains the logic to pull that payload from your local registry during the boot process. |
| **Can I change the IP after install?** | No. Static IPs in SNO are difficult to change post-install. If an IP change is required, it is usually faster to re-generate the `agent-config.yaml` and re-deploy the ISO. |
| **How do I reset the install?** | If the install fails, you must wipe the target disk (standard `dd` or `wipefs` on the partitions) and reboot from the ISO to start fresh. |

---

## Architectural Justifications & Reference Notes

| Category | Technical Requirement Details | Documentation Source |
| --- | --- | --- |
| **Gathering Logs** | If an install fails completely, use `openshift-install agent gather-bootstrap-logs --dir ./sno-config` to pull a diagnostic bundle for offline analysis. | [Troubleshooting Installation Issues](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/validation_and_troubleshooting/installing-troubleshooting) |
| **Disk Cleanup** | Before a re-install, ensure no "stale" partitions exist. The Agent installer may fail if it detects existing OVN or etcd data on the target drive. | [Agent-based Installer Guide](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_an_on-premise_cluster_with_the_agent-based_installer/index) |
| **Registry Performance** | If the image pull is extremely slow, check the CPU/Memory usage of the Quay container on the Bastion. Small bastions often throttle the registry during the initial 150GB burst. | [Mirror Registry Guide](https://www.google.com/search?q=https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/disconnected_installation_mirroring/installing-mirroring-creating-mirror-registry) |

---
