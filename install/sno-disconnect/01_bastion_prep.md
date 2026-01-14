# Step 1: Staging the Connected Bastion

Created Date: January 14, 2026
Status: Environment Preparation

The **Connected Bastion** serves as the staging area for the entire deployment. It is the only machine with internet access, used to download the OpenShift binaries and mirror the container images required for the air-gapped installation.

---

## Infrastructure Requirements

| Component | Requirement | Role |
| --- | --- | --- |
| OS | RHEL 8.x or 9.x | Base operating system for mirroring tools |
| Storage | 200 GB+ | Local cache for mirrored container images |
| Account | Red Hat Customer Portal | Access to registry.redhat.io and pull secrets |
| Tooling | oc, openshift-install, oc-mirror | Primary binaries for cluster orchestration |

---

## Binary Acquisition & Verification

Execute these commands to download the OpenShift 4.16.x binaries. Verifying the checksums is critical to ensure the integrity of the installer before it is moved into a high-security environment.

```bash
# 1. Download the OpenShift Client (oc) and Installer
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest-4.16/openshift-client-linux.tar.gz
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest-4.16/openshift-install-linux.tar.gz

# 2. Download the checksum file
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest-4.16/sha256sum.txt

# 3. Verify the integrity of the binaries
sha256sum --check --ignore-missing sha256sum.txt

# 4. Extract and move to a persistent path
tar -xvf openshift-client-linux.tar.gz
tar -xvf openshift-install-linux.tar.gz
sudo mv oc kubectl openshift-install /usr/local/bin/

# 5. Verify the binaries are in your PATH and functional
oc version --client
openshift-install version

```

---

## oc-mirror Plugin Installation (v2)

The `oc-mirror` plugin is required for the declarative mirroring workflow used in Step 2.

```bash
# 1. Download the oc-mirror v2 plugin
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest-4.16/oc-mirror.tar.gz

# 2. Extract and install
tar -xvf oc-mirror.tar.gz
chmod +x oc-mirror
sudo mv oc-mirror /usr/local/bin/

# 3. Verify installation
oc-mirror version

```

---

## Environment Persistence

To ensure the tooling is available across all terminal sessions on the Bastion, verify the following configuration.

| Task | Action | Verification |
| --- | --- | --- |
| Path Persistence | Ensure /usr/local/bin is in your secure_path | echo $PATH |
| Alias Setup | Optional: Alias k=kubectl for efficiency | alias k |
| Profile Loading | Reload .bashrc after manual path changes | source ~/.bashrc |

---

## Architectural Justifications & Reference Notes

| Category | Technical Requirement Details | Documentation Source |
| --- | --- | --- |
| Binary Choice | The client (oc) version must be equal to or newer than the target cluster version (4.16). | [OCP CLI Installation](https://www.google.com/search?q=https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/cli_tools/openshift-cli-oc-installation) |
| Checksum Verification | Integrity checks prevent the introduction of corrupted or tampered binaries into the air-gap. | [OCP Installing on Bare Metal](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_on_bare_metal/index) |
| oc-mirror v2 | Version 2 of the plugin provides a more stable, declarative approach to mirroring compared to v1. | [oc-mirror v2 Documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/disconnected_installation_mirroring/index) |

---
