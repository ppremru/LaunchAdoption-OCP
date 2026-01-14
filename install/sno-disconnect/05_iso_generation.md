# Step 5: ISO Generation & Hardware Boot

Created Date: January 14, 2026
Status: Physical Provisioning

The final phase of the installation process involves converting the YAML configurations into a bootable `agent.iso`. This file is self-contained, meaning it includes all the logic required to reach out to your local registry and provision the Single Node OpenShift cluster.

---

## Generating the Image (Step 5.1)

Run the installer from your `sno-config` directory. This command consumes the `install-config.yaml` and `agent-config.yaml` files.

```bash
# 1. Generate the bootable agent.iso
openshift-install agent create image --dir ./sno-config

# 2. Verify the image creation in the directory
ls -lh ./sno-config/agent.iso

```

---

## Media Verification & Transfer

Before booting the target hardware, ensure the ISO has been transferred to the boot media (USB or Virtual Media) without corruption.

| Verification Task | Command / Action | Importance |
| --- | --- | --- |
| ISO Checksum | sha256sum ./sno-config/agent.iso | Detects local filesystem corruption before the transfer |
| Media Integrity | Compare checksum of the ISO on the USB to the source | Prevents boot failures due to faulty physical hardware |
| Boot Mode | Ensure BIOS is set to UEFI (unless using Legacy) | Alignment with the RHCOS partition table requirement |

---

## Sensitive Data & Cleanup

The generation process creates several metadata files in the `./sno-config` directory. These files contain sensitive credentials that must be managed according to your security posture.

| File Path | Sensitivity Level | Justification |
| --- | --- | --- |
| auth/kubeadmin-password | High | Initial cluster administrator password |
| auth/kubeconfig | High | Full administrator access to the cluster API |
| metadata.json | Medium | Contains cluster ID and infrastructure metadata |

---

## Initiating the Boot (Step 5.2)

Mount the `agent.iso` via the server's Out-of-Band management (iDRAC/iLO) or insert the physical USB. Once the server starts, it will automatically initiate the "Agent" flow.

```bash
# 1. Monitor the installation progress from the Bastion
export KUBECONFIG=./sno-config/auth/kubeconfig
openshift-install agent wait-for install-complete --dir ./sno-config

```

---

## Architectural Justifications & Reference Notes

| Category | Technical Requirement Details | Documentation Source |
| --- | --- | --- |
| Self-Contained ISO | The agent.iso includes the 'Ignition' configuration. Once booted, it requires no further manual input from the administrator. | [Agent-based Installer Guide](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_an_on-premise_cluster_with_the_agent-based_installer/index) |
| ISO Verification | Corrupted ISOs often manifest as 'Kernel Panic' or 'SquashFS errors' mid-boot; verification saves hours of physical troubleshooting. | [OCP Installing on Bare Metal](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_on_bare_metal/index) |
| credential-cleanup | After the cluster is verified 'Ready', the local sno-config directory should be archived or secured to prevent unauthorized access. | [Security and Compliance Guide](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/security_and_compliance/index) |
| wait-for-completion | This command tracks the transition from the bootstrap phase to the final production state of the SNO control plane. | [SNO Installing on a single node](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_on_a_single_node/index) |

---
