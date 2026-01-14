# Step 3: Local Registry & Ingestion

Created Date: January 14, 2026
Status: Registry Configuration

Once the physical media has been moved to the air-gapped environment, the mirrored content must be "ingested" into a local registry. This registry becomes the authoritative source of truth for the SNO node, replacing the need for an internet connection to Red Hat’s servers.

---

## Registry Deployment (Step 3.1)

We utilize the Red Hat mirror-registry tool to deploy a local Quay instance on the Disconnected Bastion.

```bash
# 1. Install the mirror-registry on the air-gapped host
./mirror-registry install --quayHostname <registry_fqdn> --quayRoot /opt/quay

# 2. Add the Registry CA to the Bastion's trusted store
sudo cp /opt/quay/quay-install/quay-config/root-ca.crt /etc/pki/ca-trust/source/anchors/
sudo update-ca-trust extract

```

---

## Data Ingestion (Step 3.2)

Use the same `oc-mirror` plugin used in Step 2 to upload the images from your physical media into the local Quay registry.

```bash
# 1. Execute the ingestion from media to the local registry
oc-mirror --from ./mirror-data docker://<registry_fqdn>:8443

```

---

## Credential & Trust Hardening

This is a critical phase where you combine the default Red Hat pull secret with your new local registry credentials to create a unified secret for the installer.

| Task | Action | Verification |
| --- | --- | --- |
| Pull Secret Merge | Use 'jq' or a text editor to add the local registry auth to the Red Hat pull-secret.json | cat pull-secret.json |
| Certificate Bundle | Ensure the additionalTrustBundle contains the full CA chain (Intermediate + Root) | openssl x509 -text |
| Auth Verification | Run 'podman login' to verify the merged secret works against the local registry | podman login <registry_fqdn>:8443 |

### Pull Secret Merge Logic

The resulting `pull-secret.json` must contain both the original Red Hat auths and the new local registry auth. A JSON syntax error here is a common cause of installation failure.

```json
{
  "auths": {
    "cloud.redhat.com": { "auth": "...", "email": "..." },
    "quay.io": { "auth": "...", "email": "..." },
    "registry.redhat.io": { "auth": "...", "email": "..." },
    "<registry_fqdn>:8443": { "auth": "BASE64_ENCODED_CREDENTIALS" }
  }
}

```

---

## Architectural Justifications & Reference Notes

| Category | Technical Requirement Details | Documentation Source |
| --- | --- | --- |
| Pull Secret Integrity | The installer requires a single unified pull secret. A manual merge must maintain valid JSON structure for the node to pull images. | [Mirroring images for disconnected installation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/disconnected_installation_mirroring/index) |
| Certificate Chains | If using an organization-issued certificate, the ssl.crt must include the full chain. If the intermediate is missing, the node will reject the registry. | [Creating a mirror registry](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/disconnected_installation_mirroring/installing-mirroring-creating-mirror-registry) |
| Registry Persistence | The local registry must remain active for the life of the cluster. If the Quay service stops, the cluster will lose its ability to scale or recover. | [Disconnected installation overview](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/disconnected_installation_mirroring/index) |
| Trust Bundle Injection | The additionalTrustBundle in install-config.yaml allows the RHCOS operating system to trust your internal registry during the bootstrap phase. | [SNO Preparing to Install](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_on_a_single_node/installing-sno-preparing-to-install-on-a-single-node) |

---
