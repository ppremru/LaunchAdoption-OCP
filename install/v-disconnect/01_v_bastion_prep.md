# Step 1: Staging the Connected Bastion

Created Date: June 23, 2026
Status: Environment Preparation

The **Connected Bastion** serves as the primary staging area and software logistics checkpoint for the entire deployment. It is the only machine with outbound internet access, utilized to download authenticated OpenShift orchestrator binaries and cache the container image layers required for the secure data center installation.

---

## Infrastructure Requirements

Before executing downloads, ensure the connected staging host satisfies these baseline capacity requirements:

| Component | Requirement | Technical Operational Role |
| --- | --- | --- |
| Operating System | RHEL 8.x or 9.x | Foundational OS supporting enterprise container mirroring engines. |
| Local Storage | 500 GB+ available | Allocated space on the active mirror partition to prevent mid-stream workspace exhaustion. |
| Account Access | Red Hat Customer Portal | Valid subscription access to pull assets from registry.redhat.io. |
| Orchestration Tooling | `oc`, `openshift-install`, `oc-mirror` | Standard binaries required to compile, verify, and synchronize the deployment. |

---

## Binary Acquisition & Verification

Execute the following commands to download the standard multi-node OpenShift 4.16.x installation binaries. Cryptographic checksum verification must be completed successfully to ensure software supply-chain integrity before moving any assets across the air-gap perimeter.

> **NOTE:** Do not copy and paste these commands directly. This block is intended solely as a conceptual example of the verification workflow and must be manually tailored to your environment.

```bash
# 1. Download the standard OpenShift Client (oc) and Multi-Node Installer binaries
wget [https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest-4.16/openshift-client-linux.tar.gz](https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest-4.16/openshift-client-linux.tar.gz)
wget [https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest-4.16/openshift-install-linux.tar.gz](https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest-4.16/openshift-install-linux.tar.gz)

# 2. Retrieve the official Red Hat checksum digest signature
wget [https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest-4.16/sha256sum.txt](https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest-4.16/sha256sum.txt)

# 3. Mathematically audit binary payload integrity to prevent supply chain corruption
sha256sum --check --ignore-missing sha256sum.txt

# 4. Extract verified binaries and export into the system execution path
tar -xvf openshift-client-linux.tar.gz
tar -xvf openshift-install-linux.tar.gz
sudo mv oc kubectl openshift-install /usr/local/bin/

# 5. Confirm system path registration and active execution status
oc version --client
openshift-install version
```

---

## oc-mirror Plugin Activation (v2)

The modern, stateful declarative mirroring utility (`oc-mirror` v2) must be injected into the local orchestration paths to facilitate the data collection phase in Step 2.

> **NOTE:** Do not copy and paste these commands directly. This block is an abstract reference sample intended to show the foundational plugin setup sequence.

```bash
# 1. Download the v2 engine plugin archive
wget [https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest-4.16/oc-mirror.tar.gz](https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest-4.16/oc-mirror.tar.gz)

# 2. Unpack the binary, set execution permissions, and mount to system paths
tar -xvf oc-mirror.tar.gz
chmod +x oc-mirror
sudo mv oc-mirror /usr/local/bin/

# 3. Audit active plugin versioning using the v2 execution flag
oc-mirror version --v2
```

---

## Credential Fusion & Trust Hardening

To establish an uncompromised chain of custody, the default Red Hat subscription tokens must be fused with your secure offline registry endpoints. This produces a single, unified authorization token mandatory for both the `oc-mirror` engine and the target cluster hosts.

### Merged Pull Secret Architecture
The final `pull-secret.json` structure must match this layout. A simple syntax anomaly or missing nesting block within this JSON schema will block the core node deployment engines from initializing image lookups.

> **NOTE:** Do not copy and paste this JSON schema. It is a baseline architectural blueprint to guide manual token verification and prevent parsing errors.

```json
{
  "auths": {
    "cloud.redhat.com": { "auth": "xxxxxxxxxxxxxxxx", "email": "admin@domain.local" },
    "quay.io": { "auth": "xxxxxxxxxxxxxxxx", "email": "admin@domain.local" },
    "registry.redhat.io": { "auth": "xxxxxxxxxxxxxxxx", "email": "admin@domain.local" },
    "<internal_registry_fqdn>:8443": { "auth": "BASE64_ENCODED_INTERNAL_CREDENTIALS" }
  }
}
```
*(Note: Replace `<internal_registry_fqdn>:8443` with the explicit, internal FQDN and target port used by your enterprise offline mirror-registry).*

---

## Architectural Justifications & Reference Notes

| Category | Technical Requirement Details | Documentation Source |
| --- | --- | --- |
| Utility Alignment | The core client binary (`oc`) version must match or exceed the target minor version of the platform (4.16) to ensure backwards API compatibility. | [OpenShift CLI Installation Guide](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/cli_tools/openshift-cli-oc-installation) |
| Border Auditing | Enforcing local SHA256 checksum verifications eliminates corrupted or tampered payloads prior to scheduling physical transport to secure zones. | [Bare-Metal Infrastructure Deployments](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_on_bare_metal/index) |
| Token Fusion | OpenShift installation architectures demand a single, unified cryptographic credential mapping table to negotiate lookups across both public and private boundaries. | [Mirroring Offline Platform Payloads](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/mirroring_images_for_a_disconnected_installation_using_the_oc-mirror_plugin/) |
| Enterprise Mirroring (v2) | The v2 engine utilizes structured local workspace metadata to accurately calculate incremental graph changes during future Day 2 updates. | [oc-mirror Engine Specifications](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/mirroring_images_for_a_disconnected_installation_using_the_oc-mirror_plugin/) |
