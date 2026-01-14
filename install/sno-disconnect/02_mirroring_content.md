# Step 2: Mirroring Content for the Air-Gap

Created Date: January 14, 2026
Status: Data Collection

In a disconnected environment, you must manually "mirror" all required container images from Red Hat’s registries to local media. This process ensures that every component needed for the SNO installation is physically present before moving to the air-gapped site.

---

## Storage & Workspace Verification

Before beginning the mirror, ensure the Bastion has sufficient local storage. The mirroring process requires a significant workspace for metadata and image layers.

| Category | Requirement | Verification Command |
| --- | --- | --- |
| Disk Space | 200 GB+ available on the mirroring partition | df -h |
| Workspace | Write permissions for the oc-mirror-workspace directory | ls -ld . |
| Backend | Physical media (SSD/HDD) formatted and mounted | lsblk |

---

## ImageSetConfiguration (Step 2.1)

The `imageset-config.yaml` file defines exactly which versions of OpenShift and which Operators will be mirrored.

```yaml
kind: ImageSetConfiguration
apiVersion: mirror.openshift.io/v2alpha1
mirror:
  platform:
    channels:
    - name: stable-4.16
      type: ocp
      minVersion: 4.16.0
      maxVersion: 4.16.0
  operators:
    - catalog: registry.redhat.io/redhat/redhat-operator-index:v4.16
      packages:
        - name: lvms-operator
        - name: cluster-logging
  additionalImages:
    - name: registry.redhat.io/ubi9/ubi:latest

```

---

## Executing the Mirror (Step 2.2)

Use the `oc-mirror` plugin to pull the images. It is highly recommended to perform a dry run first to validate the configuration file without downloading data.

```bash
# 1. Perform a dry run to validate the ImageSetConfiguration
oc-mirror --config ./imageset-config.yaml file://./mirror-data --dry-run

# 2. Execute the actual mirror to local storage
oc-mirror --config ./imageset-config.yaml file://./mirror-data

# 3. Verify the generated 'mirror-data' directory contains the 'v2' metadata
ls -R ./mirror-data

```

---

## Architectural Justifications & Reference Notes

| Category | Technical Requirement Details | Documentation Source |
| --- | --- | --- |
| Workspace Capacity | The oc-mirror tool stores metadata and blob layers in the local workspace; insufficient space will cause a mid-process failure. | [Mirroring images for disconnected installation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/disconnected_installation_mirroring/index) |
| Dry Run Validation | Using the --dry-run flag catches syntax errors in the YAML and confirms package availability before committing to a massive download. | [oc-mirror v2 Documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/disconnected_installation_mirroring/index) |
| Platform Pinning | Setting minVersion and maxVersion to the same value (4.16.0) prevents the tool from mirroring every version in the stable channel. | [oc-mirror Reference](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/disconnected_installation_mirroring/index) |
| Operator Filtering | By listing specific packages (lvms, logging), you reduce the final mirror size by gigabytes compared to mirroring the full catalog. | [Filtering operator catalogs](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/disconnected_installation_mirroring/index) |

---
