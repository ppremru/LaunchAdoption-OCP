# Step 3: Registry Setup (Disconnected)

Created Date: January 14, 2026

Status: Ingestion Phase

This document guides the administrator through setting up the local mirror registry on the **Disconnected Bastion** and performing the **Ingestion** phase of the Sneakernet process. This registry will serve as the single source of truth for the SNO node.

## Local Registry Overview

| Task | Description |
| --- | --- |
| **Mirror Registry Installation** | Deploying the `mirror-registry` tool (Quay) onto the Disconnected Bastion to host container images. |
| **Trust Establishment** | Configuring the Disconnected Bastion and future SNO node to trust the local registry's self-signed or internal CA certificate. |
| **Content Ingestion** | Uploading the mirrored image set from physical media into the local registry using `oc-mirror`. |

### Registry Configuration Reference Script

> **Note**: The following commands represent a **baseline implementation** for a local Quay registry. Ensure your physical media is mounted to the Disconnected Bastion before starting.

```bash
# 1. Install the Mirror Registry (Quay)
# Ensure the mirror-registry.tar.gz is extracted
./mirror-registry install --unattended \
                          --targetHostname <registry_fqdn> \
                          --targetAddr <registry_ip>

# 2. Add Registry CA to Trusted Bundle
# This ensures oc-mirror can authenticate without TLS errors
sudo cp /etc/pki/ca-trust/source/anchors/local-registry.crt /etc/pki/ca-trust/source/anchors/
sudo update-ca-trust

# 3. Perform the Ingestion (Media to Registry)
# Replace <path_to_media> with your mount point
oc-mirror --from file://<path_to_media>/mirror-data \
          docker://<registry_fqdn>:8443 \
          --workspace file://./workspace

```

---

## Technical Process & Justifications

| Category | Justification |
| --- | --- |
| **1. Unattended Install** | Using the `--unattended` flag ensures a consistent deployment and automatically generates the `quay-install.log` for troubleshooting. |
| **2. Port 8443** | By default, the `mirror-registry` tool uses port 8443 for HTTPS traffic; ensure this port is open in the local firewall. |
| **3. Workspace Sync** | You must use the same `workspace` folder (or metadata) that was generated during the **Collection** phase to ensure successful ingestion. |

---

## Architectural Justifications & Reference Notes

| Category | Technical Requirement Details | Documentation Source |
| --- | --- | --- |
| **Internal DNS** | The `<registry_fqdn>` used during installation must be resolvable by both the Disconnected Bastion and the Target SNO Node. | [Installing on a single node](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_on_a_single_node/index) |
| **Pull Secret Merge** | After the ingestion, `oc-mirror` generates a new `pull-secret.json`. This **must** be merged with your original Red Hat pull secret to allow the SNO node to pull from the local registry. | [oc-mirror Documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/disconnected_installation_mirroring/installing-mirroring-disconnected) |
| **Persistent Storage** | The registry data is stored in `/var/lib/quay` by default. Ensure this directory is backed by sufficient persistent storage (minimum 200GB). | [Creating a mirror registry](https://www.google.com/search?q=https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/disconnected_installation_mirroring/installing-mirroring-creating-mirror-registry) |

---