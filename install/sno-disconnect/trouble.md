# Troubleshooting & FAQ

Created Date: January 14, 2026
Status: Maintenance & Support

Deployment in disconnected environments introduces unique failure points related to certificate trust, DNS resolution, and image mirroring. This guide covers the most common issues encountered when using the Agent-based Installer for Single Node OpenShift (SNO).

---

## Agent Boot & Installation Logs

If the `wait-for install-complete` command hangs or the node fails to reach the "Ready" state, you must inspect the logs directly from the SNO node via SSH or the local console.

| Log Source | Command | Purpose |
| --- | --- | --- |
| Agent Installer | journalctl -u agent-installer -f | Primary log for the initial bootstrap and image pull |
| Assisted Service | journalctl -u assisted-service | Tracks the orchestration of the installation steps |
| Kubelet | journalctl -u kubelet -f | Monitors the health of the Kubernetes node agent |
| Pod Logs | oc logs -n <namespace> <pod_name> | Diagnostic data for specific system operators |

---

## Common Failure Scenarios

| Symptom | Probable Cause | Resolution |
| --- | --- | --- |
| ImagePullBackOff | Registry trust failure | Ensure the Registry CA is in additionalTrustBundle |
| ImagePullBackOff | Malformed pull secret | Verify JSON structure of the merged pull-secret.json |
| Node Not Found | DNS resolution failure | Verify api.<cluster>.<domain> points to the SNO IP |
| etcd Degraded | Time drift or disk latency | Check chronyd status and disk I/O wait times |
| Static IP Missing | Interface name mismatch | Ensure agent-config.yaml matches the kernel device name |

---

## Certificate & Time Drift Management

In a disconnected environment, the lack of an external time source can cause the cluster’s internal certificate authority to drift, leading to cluster-wide authentication failures.

* **Clock Skew**: If the SNO node clock differs from the Registry Bastion by more than a few seconds, the bootstrap process may reject the Registry's TLS certificate.
* **Certificate Expiry**: If the SNO node is powered down for more than 30 days, the Kubelet certificates may expire. Upon reboot, you may need to manually approve the CSRs (`oc get csr`) to restore node connectivity.
* **NTP Recovery**: If `chronyd` cannot reach the local NTP server, manually set the date on the SNO node using `date -s "YYYY-MM-DD HH:MM:SS"` to allow initial certificate validation to proceed.

---

## Architectural Justifications & Reference Notes

| Category | Technical Requirement Details | Documentation Source |
| --- | --- | --- |
| Log Access | During the bootstrap phase, the standard 'oc' commands may not work. Accessing logs via 'journalctl' is the only way to debug pull failures. | [OCP Troubleshooting Guide](https://www.google.com/search?q=https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/support/troubleshooting-installations) |
| Certificate Drift | In air-gapped sites, local NTP is the heartbeat of the cluster. Without it, the etcd quorum and API security will eventually collapse. | [SNO Performance and Scalability](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/scalability_and_performance/index) |
| Pull Secret Logic | A single typo in the BASE64 string of a manually merged pull secret will prevent the node from authenticated with the local Quay registry. | [Disconnected installation mirroring](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/disconnected_installation_mirroring/index) |
| Interface Predictability | RHCOS uses predictable network naming. If the BIOS or a hardware change alters the naming (e.g., eno1 to eno2), the Agent config will fail. | [Agent-based Installer Guide](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_an_on-premise_cluster_with_the_agent-based_installer/index) |

---
