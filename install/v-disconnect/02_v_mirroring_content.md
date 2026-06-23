# Step 2: Content Mirroring for the Air-Gap

Created Date: June 23, 2026
Status: Data Collection

To establish a functioning software logistics pipeline, all required container image layers must be bundled onto local media before crossing the air-gap perimeter. In an enterprise virtualization deployment, the core cluster platform images and the complete virtualization operator ecosystem must be synchronized simultaneously. This locks their inter-operator version dependencies and prevents broken mapping definitions when the cluster is birthed natively offline.

---

## Pre-Mirror Workspace Validation

Prior to commencing the data pull, verify that the connected staging host satisfies these baseline workspace constraints to prevent mid-stream download exhaustion:

| Category | Verification Requirement | Inspection Command |
| --- | --- | --- |
| Storage Capacity | 500 GB+ available on the active mirroring partition | `df -h` |
| Folder Permissions | Write access confirmed on the local mirror workspace directory | `ls -ld .` |
| Transport Media | Physical media (SSD/HDD) formatted and correctly mounted | `lsblk` |

---

## ImageSetConfiguration Blueprint (Step 2.1)

The blueprint configuration defines the specific platform channels and filtered operator catalogs that the mirror engine will collect. 

> **NOTE:** Do not copy and paste this configuration file directly. This block is intended solely as a conceptual example of a unified schema blueprint and must be manually adjusted to reflect your targeted minor version streams and internal registry parameters.

```yaml
kind: ImageSetConfiguration
apiVersion: mirror.openshift.io/v2alpha1
mirror:
  platform:
    channels:
    - name: stable-4.16
  operators:
    - catalog: registry.redhat.io/redhat/redhat-operator-index:v4.16
      packages:
        - name: kubevirt-hyperconverged
          channels:
          - name: stable
        - name: kubernetes-nmstate
          channels:
          - name: stable
```

---

## Executing the Mirror Payload (Step 2.2)

Utilize the stateful `oc-mirror` plugin engine to process the configuration blueprint. A strict dry-run simulation must be completed first to catch structural errors before initiating the full software download.

> **NOTE:** Do not copy and paste these commands directly. This block is an abstract reference sample showing the data collection syntax and must be manually tailored to your local path variables.

```bash
# 1. Execute a dry run utilizing the required explicit v2 flag to validate the schema
oc-mirror --config ./imageset-config.yaml file://./mirror-data --dry-run --v2

# 2. Execute the actual data collection payload onto local storage disk cache
oc-mirror --config ./imageset-config.yaml file://./mirror-data --v2

# 3. Confirm the generated output bundle contains the structured metadata layers
ls -R ./mirror-data
```

---

## Architectural Justifications & Reference Notes

| Category | Technical Requirement Details | Documentation Source |
| --- | --- | --- |
| Workspace Capacity | The `oc-mirror` tool stores metadata and blob layers locally during compilation; running out of block disk space during execution corrupts the local state database. | [Mirroring Images for Disconnected Environments](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/mirroring_images_for_a_disconnected_installation_using_the_oc-mirror_plugin/) |
| Dry Run Controls | Simulating the execution via the `--dry-run` flag identifies syntax deviations or unavailable packages before committing transport infrastructure. | [oc-mirror Plugin Operations](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/mirroring_images_for_a_disconnected_installation_using_the_oc-mirror_plugin/) |
| Channel Over Pinning | Defining a standard release channel segment instead of forcing a fixed micro-version ensures the local warehouse registry acquires the dependency graph nodes required for future minor cluster updates. | [Configuring ImageSet Repositories](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/mirroring_images_for_a_disconnected_installation_using_the_oc-mirror_plugin/) |
| Unified Bundling Strategy | Declaring `kubevirt-hyperconverged` and `kubernetes-nmstate` inside the same configuration locks
