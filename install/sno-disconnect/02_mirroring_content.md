# Step 2: Mirroring Content (Connected)

Created Date: January 14, 2026

Status: Content Preparation

This document guides the administrator through creating the declarative `ImageSetConfiguration` file and executing the initial mirror to physical media. This represents the **Collection** phase of the Sneakernet process.

## Mirroring Configuration Overview

| Task | Description |
| --- | --- |
| ImageSetConfiguration | A YAML file that defines exactly which OpenShift versions, operators, and additional images will be mirrored. |
| Storage Backend | For the initial mirror to physical media, a local directory is used as the temporary backend to hold the metadata and image chunks. |
| Mirror Execution | The process of pulling data from Red Hat registries and packaging it for transfer to the air-gapped environment. |

### Mirroring Preparation Reference Script

> **Note**: The following YAML and commands are a **functional guide**. Ensure your physical media is mounted and has at least 200GB of free space before starting.

```yaml
# 1. Create the ImageSetConfiguration YAML
# File: imageset-config.yaml
# Note: minVersion and maxVersion are used to pin the specific release.
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

```bash
# 2. Execute the Mirror to Local Directory
# Replace <path_to_removable_media> with your actual mount point.
oc-mirror --config ./imageset-config.yaml \
          file://<path_to_removable_media>/mirror-data \
          --workspace file://./workspace

```

---

## Technical Process & Justifications

| Category | Justification |
| --- | --- |
| Version Pinning | Using `minVersion` and `maxVersion` ensures you only download the specific 4.16.0 payload, saving significant disk space and transfer time. |
| Workspace Management| The `--workspace` flag tracks metadata; keeping this folder is critical for performing future incremental updates (Z-streams). |
| Operator Selection | Including `lvms-operator` is a common best practice for SNO to manage local disk storage effectively without a SAN. |

---

## Architectural Justifications & Reference Notes

| Category | Technical Requirement Details | Documentation Source |
| --- | --- | --- |
| Platform Mirroring | The platform mirror includes the RHCOS (Red Hat Enterprise Linux CoreOS) images required for the Agent-based installer to build the ISO. | [Mirroring images for disconnected installation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/disconnected_installation_mirroring/index) |
| Metadata Integrity | When the mirror completes, a `metadata.json` file is created. This file **must** be transferred with the image data to the disconnected environment to ensure successful synchronization. | [oc-mirror Documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/disconnected_installation_mirroring/installing-mirroring-disconnected) |
| Transfer Validation | After mirroring to physical media, it is highly recommended to perform a `du -sh` on the output folder to ensure the size matches the expected payload (~100-150GB). | [Disconnected Mirroring Guide](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/disconnected_installation_mirroring/index) |

---
