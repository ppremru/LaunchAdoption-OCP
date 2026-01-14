# Day 2: Post-Installation & Operational Health

Created Date: January 14, 2026
Status: Post-Install / Day 2 Operations

Once the Single Node OpenShift (SNO) cluster is "Ready," the focus shifts to operationalizing the environment. In a disconnected solution, this involves finalizing local storage for persistent data and configuring cluster logging for auditability.

## Day 2 Operational Overview

| Task | Description |
| --- | --- |
| Storage Provisioning | Configuring the **LVM Storage (LVMS)** operator to provide persistent volumes (PVs) using the remaining local disk space. |
| Audit & Logging | Deploying the **Cluster Logging** operator to aggregate system and application logs for troubleshooting and compliance. |
| Cert-Manager Setup | (Optional) Deploying cert-manager to automate the management and issuance of certificates within the cluster. |

### Day 2 Reference Script

> **Note**: These commands assume the required operators were included in your initial mirror (Step 2). Ensure your `KUBECONFIG` is still exported.

```bash
# 1. Verify CatalogSource Initialization
# CRITICAL: The local catalog MUST show 'READY' before proceeding. 
# If it shows 'PENDING' or 'IMAGEPULLBACKOFF', check your Registry CA trust.
oc get catalogsource -n openshift-marketplace

# 2. Prepare Namespaces
# Operators require specific namespaces to be created before configuration
oc create namespace openshift-storage
oc create namespace openshift-logging

# 3. Verify Operator Availability
# Once the catalog is ready, ensure the mirrored operators are visible in the hub
oc get packagemanifests -n openshift-marketplace | grep -E 'lvms|logging'

# 4. Configure Local Storage (LVMS)
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
          overprovisionRatio: 10
          sizePercent: 90
EOF

# 5. Verify Storage Readiness
# Wait 1-2 minutes, then verify the StorageClass is created and marked as default
oc get sc

```

---

## Architectural Justifications & Reference Notes

| Category | Technical Requirement Details | Documentation Source |
| --- | --- | --- |
| CatalogSource Status | In air-gapped sites, the `CatalogSource` can take several minutes to pull the index image from your local registry. Installation of operators will fail if the status is not `READY`. | [oc-mirror Documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/disconnected_installation_mirroring/installing-mirroring-disconnected) |
| Namespace Scoping | Operators like Logging and Storage should be isolated in their own namespaces (`openshift-logging`, `openshift-storage`) to maintain resource boundaries. | [LVMS Operator Documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/storage/logical-volume-manager-storage) |
| Disk Readiness | For LVMS to work, the target disk must be "clean" (no existing partitions or filesystems) outside of the main OS partition. | [LVMS Operator Documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/storage/logical-volume-manager-storage) |
| Resource Constraints | SNO nodes often have limited CPU/RAM. Monitor the overhead of the Logging operator, as it can consume significant resources during high-burst log events. | [SNO Performance and Scalability](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/scalability_and_performance/index) |
| Disconnected Updates | Any future "Day 2" operators must be mirrored via the same `oc-mirror` process established in Step 2 to maintain the air-gapped supply chain integrity. | [oc-mirror Documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/disconnected_installation_mirroring/installing-mirroring-disconnected) |

---

**Next Step:** Since we've finalized the Day 2 operations, is there any other file in the repository you'd like me to review for "Secret Sauce" improvements or technical gaps?
