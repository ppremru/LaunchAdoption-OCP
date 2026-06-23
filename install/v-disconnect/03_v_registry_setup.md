# Step 3: Local Registry & Ingestion

Created Date: June 23, 2026
Status: Registry Configuration

Once the physical transport media has safely crossed the secure network perimeter, the mirrored container images must be "ingested" into your internal container registry. This registry functions as your local software warehouse, completely replacing public internet endpoints as the sole authoritative source of truth for the multi-node cluster.

---

## Registry Operational Verification

Prior to pushing the data payload, ensure the target disconnected host and local OCI registry infrastructure satisfy these operational readiness checkpoints:

| Category | Verification Requirement | Inspection Method / Command |
| --- | --- | --- |
| Storage Allocation | 500 GB+ available on the destination registry storage volume | `df -h` |
| Local FQDN Resolution | Registry FQDN resolves correctly across the offline server pools | `nslookup <registry_fqdn>` |
| Transport Media Integrity | Payload data directory matches the layer layout generated in Step 2 | `ls -R ./mirror-data` |

---

## Registry Deployment (Step 3.1)

In environments utilizing the standard Red Hat mirror-registry enterprise tool, a localized Quay engine is deployed directly onto the disconnected bastion host.

> **NOTE:** Do not copy and paste these commands directly. This block represents a conceptual framework for local registry initialization and certificate anchoring; parameters must be manually customized to match your specific corporate domain name and path rules.

```bash
# 1. Example syntax for executing the mirror-registry automated installer tool
./mirror-registry install --quayHostname <registry_fqdn> --quayRoot /opt/quay

# 2. Reference workflow for appending the local registry root CA to the OS trust store
sudo cp /opt/quay/quay-install/quay-config/root-ca.crt /etc/pki/ca-trust/source/anchors/
sudo update-ca-trust extract
```

---

## Data Ingestion Execution (Step 3.2)

Utilize the `oc-mirror` plugin tool inside the offline network to upload the structured image layers from your removable storage media directly into the internal OCI registry.

> **NOTE:** Do not copy and paste these commands directly. This block is an abstract reference model for launching the ingestion phase and must be edited to declare the correct 8443 enterprise mirror-registry listening port.

```bash
# 1. Sample syntax for processing and uploading the offline payload bundle from local media
oc-mirror --from ./mirror-data docker://<registry_fqdn>:8443 --v2
```

---

## Credential Fusion & Trust Hardening

To establish an uncompromised chain of custody, the local warehouse registry authentication keys must be securely merged with your existing Red Hat cloud portal tokens.

### Pre-Flight Token Validation Checkpoints
Before proceeding to generate cluster installer configurations, manually confirm the following trust matrices:

| Validation Task | Execution Requirement | Success Verification Check |
| --- | --- | --- |
| Pull Secret Merge | Append the internal registry auth block to the `pull-secret.json` structure using a text editor or `jq`. | Verify file output is clear of formatting anomalies using a JSON validator tool. |
| Trust Chain Extraction | Confirm the exported internal registry CA file contains the complete, uninterrupted certificate chain. | `openssl x509 -text -noout -in root-ca.crt` |
| Authentication Pass Gate | Execute a container runtime login to prove the cluster nodes can successfully pull from the warehouse. | `podman login --authfile pull-secret.json <registry_fqdn>:8443` |

### Fused Pull Secret Schema Blueprint
The merged credential matrix must preserve a valid, flat JSON hierarchy. 

> **NOTE:** Do not copy and paste this JSON schema. It is a baseline structural design blueprint intended strictly for manual token structure audit and comparison purposes.

```json
{
  "auths": {
    "cloud.redhat.com": { "auth": "xxxxxxxxxxxxxxxx", "email": "admin@domain.local" },
    "quay.io": { "auth": "xxxxxxxxxxxxxxxx", "email": "admin@domain.local" },
    "registry.redhat.io": { "auth": "xxxxxxxxxxxxxxxx", "email": "admin@domain.local" },
    "<registry_fqdn>:8443": { "auth": "BASE64_ENCODED_INTERNAL_CREDENTIALS" }
  }
}
```

---

## Architectural Justifications & Reference Notes

| Category | Technical Requirement Details | Documentation Source |
| --- | --- | --- |
| Pull Secret Schema Integrity | The cluster deployment engine requires a single, unified string object. A manual merge error or syntax drift will break container lookup routines during early node creation. | [Mirroring Images for Disconnected Installations](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/mirroring_images_for_a_disconnected_installation_using_the_oc-mirror_plugin/) |
| Certificate Chain Retention | When leveraging an enterprise CA, the signing chain must be preserved intact. Missing intermediate certificates will cause cluster hosts to reject connection attempts during bootstrap phases. | [Creating a Disconnected Mirror Registry](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing/creating-a-mirror-registry-for-a-disconnected-installation) |
| Local Warehouse Persistence | The internal registry must be engineered as permanent infrastructure. If the OCI registry service loses availability, the operational cluster instantly loses the ability to scale, recover pods, or deploy applications. | [Disconnected Environment Overview](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing/creating-a-mirror-registry-for-a-disconnected-installation) |
| Trust Bundle Integration | Embedding the registry's root CA within the `additionalTrustBundle` field allows the Red Hat Enterprise Linux CoreOS (RHCOS) kernel to implicitly establish encrypted TLS trust before its networking stack is fully initialized. | [Preparing Infrastructure for Disconnected Installations](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_a_cluster_using_the_agent-based_installer/) |
