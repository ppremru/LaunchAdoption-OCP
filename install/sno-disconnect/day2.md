# Day 2: Post-Installation & Operational Health

Created Date: January 14, 2026

Status: Post-Install / Day 2 Operations

Once the Single Node OpenShift (SNO) cluster is "Ready," the focus shifts to operationalizing the environment. In a disconnected solution, this involves finalizing local storage for persistent data and configuring cluster logging for auditability.

## Day 2 Operational Overview

| Task | Description |
| --- | --- |
| **Storage Provisioning** | Configuring the **LVM Storage (LVMS)** operator to provide persistent volumes (PVs) using the remaining local disk space. |
| **Audit & Logging** | Deploying the **Cluster Logging** operator to aggregate system and application logs for troubleshooting and compliance. |
| **Cert-Manager Setup** | (Optional) Deploying cert-manager to automate the management and issuance of certificates within the cluster. |

### Day 2 Reference Script

> **Note**: These commands assume the required operators were included in your initial mirror (Step 2). Ensure your `KUBECONFIG` is still exported.

```bash
# 1. Verify Operator Availability
# Ensure the operators mirrored in Step 2 are visible in the cluster
oc get packagemanifests -n openshift-marketplace | grep -E 'lvms|logging'

# 2. Configure Local Storage (LVMS)
# Create the LVMCluster resource to initialize local disk provisioning
cat <<EOF | oc apply -f -
apiVersion: lvm.topolvm.io/v1alpha1
kind: LVMCluster
metadata:
  name: lvmcluster-sno
  namespace: openshift-storage
spec:
  storage:
    deviceClasses:
      - name: vg1
        thinPoolConfig:
          name: thin-pool-1
          sizePercent: 90
          overprovisionRatio: 10
EOF

# 3. Check Persistence Support
# Verify that a default StorageClass (SC) is now available for applications
oc get sc

```

---

## Technical Process & Justifications

| Category | Justification |
| --- | --- |
| **1. LVMS Operator** | SNO lacks external storage (SAN/NAS). The LVM Storage operator allows the node to use its own local NVMe/SSD disks for dynamic volume provisioning. |
| **2. Namespace Scoping** | Operators like Logging and Storage should be isolated in their own namespaces (`openshift-logging`, `openshift-storage`) to maintain resource boundaries. |
| **3. Overprovisioning** | In a single-node environment, a 10x overprovisioning ratio is often used to maximize the utility of the available physical disk for multiple small workloads. |

---

## Architectural Justifications & Reference Notes

| Category | Technical Requirement Details | Documentation Source |
| --- | --- | --- |
| **Disk Readiness** | For LVMS to work, the target disk must be "clean" (no existing partitions or filesystems) outside of the main OS partition. | [LVMS Operator Documentation](https://www.google.com/search?q=https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/storage/logical-volume-manager-storage) |
| **Resource Constraints** | SNO nodes often have limited CPU/RAM. Monitor the overhead of the Logging operator, as it can consume significant resources during high-burst log events. | [SNO Performance and Scalability](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/scalability_and_performance/index) |
| **Disconnected Updates** | Any future "Day 2" operators must be mirrored via the same `oc-mirror` process established in Step 2 to maintain the air-gapped supply chain integrity. | [oc-mirror Documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/disconnected_installation_mirroring/installing-mirroring-disconnected) |

---

**This concludes the full documentation set for your disconnected SNO 4.16 solution.** **Is there anything else I can help you with?** For example, would you like a **Troubleshooting FAQ** covering the most common air-gapped installation errors (like certificate mismatches or pull-backoff issues)?