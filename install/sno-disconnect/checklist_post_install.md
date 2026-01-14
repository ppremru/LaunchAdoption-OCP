# Post-Install Verification Checklist

Created Date: January 14, 2026

Once the `openshift-install` tool reports that the installation is complete, use this checklist to verify the operational integrity of the SNO node. In a disconnected, hardened environment, this ensures that the internal image registry, storage, and networking are correctly utilizing your local infrastructure.

---

## Core Cluster Health Verification

| Task | Command | Expected Result | Status |
| --- | --- | --- | --- |
| Node Status | `oc get nodes` | The node is in the `Ready` state | [ ] |
| Operator Status | `oc get clusteroperators` | All operators show `AVAILABLE: True` | [ ] |
| — | — | All operators show `PROGRESSING: False` | [ ] |
| — | — | All operators show `DEGRADED: False` | [ ] |
| Certificate Validity | `oc get csr` | All CSRs are `Approved,Issued` | [ ] |

---

## Disconnected Supply Chain Verification

| Check | Action | Justification | Status |
| --- | --- | --- | --- |
| Image Pulls | `oc run test-pull --image=<registry_fqdn>:8443/ubi9/ubi:latest -- sleep 10` | Confirms the node can pull images from the local Quay registry | [ ] |
| Redirection | `oc get imagecontentsourcepolicy` | Verifies the cluster is redirecting `quay.io` requests to your local registry | [ ] |
| Trust Bundle | `oc get cm user-ca-bundle -n openshift-config` | Confirms your Internal Root CA is present in the cluster trusted store | [ ] |

---

## Storage & Workload Verification

| Component | Action | Expected Result | Status |
| --- | --- | --- | --- |
| Local Storage | `oc get sc` | The `topolvm-provisioner` (LVMS) storage class is present | [ ] |
| — | — | Storage class is marked as `(default)` | [ ] |
| PV Creation | Create a test `PersistentVolumeClaim` | The PVC status changes to `Bound` | [ ] |
| Air-Gap Security | `oc exec <pod_name> -- nslookup google.com` | Should fail, confirming no unauthorized outbound routing | [ ] |

---

## Architectural Justifications & Reference Notes

| Category | Technical Requirement Details | Documentation Source |
| --- | --- | --- |
| Operator Degradation | If operators are `Degraded`, check the `openshift-image-registry` operator first; in disconnected sites, this is often caused by storage or certificate mismatches. | [Troubleshooting Installation Issues](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/validation_and_troubleshooting/installing-troubleshooting) |
| Workload Testing | Running a simple `UBI` pod is the fastest way to verify that the entire stack—registry, networking, and scheduling—is functional. | [SNO Performance and Scalability](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/scalability_and_performance/index) |
| Log Aggregation | Finalize the "Day 2" logging configuration only after confirming the cluster operators are stable to avoid resource contention during initial boot. | [Cluster Logging Documentation](https://www.google.com/search?q=https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/logging/index) |

