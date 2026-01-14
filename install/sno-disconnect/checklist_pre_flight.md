---

# Pre-Flight Deployment Checklist

Created Date: January 14, 2026
Status: Final Verification

Before executing the `agent.iso` on the physical hardware, use this checklist to ensure the air-gapped environment is fully prepared. Failure to verify these items often results in installation hangs that are difficult to debug mid-process.

---

## Environment & Infrastructure Gates

| Category | Checkpoint Item | Verification Method |
| --- | --- | --- |
| DNS | api.<cluster>.<domain> resolves to SNO IP | dig or nslookup |
| DNS | *.apps.<cluster>.<domain> resolves to SNO IP | dig or nslookup |
| DNS | <registry_fqdn> resolves to Bastion IP | dig or nslookup |
| NTP | Local NTP Server is reachable from SNO segment | ping <ntp_ip> |
| Time Sync | Bastion clock is synchronized with Local NTP | chronyc sources -v |
| Connectivity | Port 8443 is open on Disconnected Bastion | telnet <bastion_ip> 8443 |

---

## Hardware & Manifest Alignment

Ensure the physical server characteristics match the logic defined in your `agent-config.yaml` and `install-config.yaml`.

| Component | Checkpoint Item | Verification Method |
| --- | --- | --- |
| Interface | MAC Address matches agent-config.yaml exactly | Physical label or BIOS check |
| Interface | Kernel name (eno1/ens3) matches agent-config.yaml | Verified via Live ISO/Hardware spec |
| Storage | Target disk is "clean" (no existing partitions) | wipefs (if re-using hardware) |
| Resources | Minimum 8 vCPUs and 16 GB to 32 GB RAM available | BIOS/Hardware specification |
| Firmware | Boot mode set to UEFI | BIOS Boot settings |

---

## Registry & Supply Chain Integrity

| Category | Checkpoint Item | Verification Method |
| --- | --- | --- |
| Trust | Registry CA (and Intermediate) is in additionalTrustBundle | Review install-config.yaml |
| Auth | Pull Secret contains valid auth for local registry | jq .auths pull-secret.json |
| Content | oc-mirror report confirms all images are present | Review mirror metadata |
| Media | agent.iso checksum matches source on Bastion | sha256sum /dev/sdX |

---

## Architectural Justifications & Reference Notes

| Category | Technical Requirement Details | Documentation Source |
| --- | --- | --- |
| NTP Accuracy | OpenShift's etcd and certificate verification mechanisms require sub-second clock synchronization between the node and its time source. | [SNO Preparing to Install](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_on_a_single_node/installing-sno-preparing-to-install-on-a-single-node) |
| MAC/Interface | The Agent-based installer uses the MAC address to bind the static IP; if the interface name is wrong, the network stack will not initialize. | [Agent-based Installer Guide](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_an_on-premise_cluster_with_the_agent-based_installer/index) |
| Wildcard DNS | The *.apps record is required for the OpenShift Ingress Controller to route traffic to the console and user applications. | [OCP Networking Overview](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/networking/index) |
| UEFI Requirement | RHCOS uses a specific partition layout that requires UEFI firmware for modern secure boot and disk management capabilities. | [OCP Installing on Bare Metal](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_on_bare_metal/index) |

---
