---

# Post-Installation Verification Checklist

Created Date: January 14, 2026
Status: Cluster Validation

Once the `wait-for install-complete` command returns successfully, use this checklist to verify that the cluster is healthy and that the air-gapped configuration is correctly routing image requests to your local registry.

---

## Core Cluster Health

| Category | Checkpoint Item | Verification Command |
| --- | --- | --- |
| Nodes | SNO node is in Ready status | oc get nodes |
| Operators | All ClusterOperators are Available: True | oc get co |
| Version | Cluster version matches 4.16.x | oc get clusterversion |
| Certificates | No pending CSRs require approval | oc get csr |

---

## Air-Gap & Registry Validation

This section ensures the "Secret Sauce" of the disconnected installation (the image redirection) is functioning as intended.

| Category | Checkpoint Item | Verification Command |
| --- | --- | --- |
| ICSP | ImageContentSourcePolicy is present and active | oc get icsp |
| Registry | Internal image-registry operator is Available | oc get co image-registry |
| Storage | Internal registry has a valid storage backend | oc get configs.imageregistry.operator.openshift.io/cluster -o yaml |
| Redirection | Pods are pulling from <registry_fqdn> | oc get pods -A -o jsonpath='{.items[*].spec.containers[*].image}' |

---

## Persistence & Storage Readiness

| Category | Checkpoint Item | Verification Command |
| --- | --- | --- |
| StorageClass | A default StorageClass is present (after LVMS install) | oc get sc |
| PVs | Persistent Volumes can be bound to claims | oc get pvc -A |
| Etcd | Etcd is healthy and synchronized | oc get pods -n openshift-etcd |

---

## Architectural Justifications & Reference Notes

| Category | Technical Requirement Details | Documentation Source |
| --- | --- | --- |
| ICSP Role | The ImageContentSourcePolicy is the mechanism that instructs the node to transparently swap 'quay.io' links for your local registry FQDN. | [Disconnected installation mirroring](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/disconnected_installation_mirroring/index) |
| Registry Storage | In a Single Node OpenShift (SNO) deployment, the internal registry cannot use 'shared' storage like NFS easily; it must be configured for 'EmptyDir' or a local PV. | [Image Registry Operator](https://www.google.com/search?q=https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/registry/index) |
| CSR Monitoring | While the Agent-installer handles most certificate signing, manual approval of Kubelet CSRs is occasionally required if the node name changes during boot. | [OCP Node Management](https://www.google.com/search?q=https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/nodes/index) |
| etcd Quorum | On a single node, etcd acts as a standalone database. High disk I/O latency on the SNO node is the leading cause of etcd instability. | [SNO Performance and Scalability](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/scalability_and_performance/index) |

---
