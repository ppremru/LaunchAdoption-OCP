# Step 4: SNO Node Configurations

Created Date: January 14, 2026
Status: Manifest Definition

The Agent-based Installer requires two primary configuration files: `install-config.yaml` and `agent-config.yaml`. These files define the cluster's logical identity and the physical hardware networking respectively.

---

## Workspace Setup

Create a dedicated directory for the installation assets. This workspace will house the manifests and eventually the generated metadata.

```bash
mkdir sno-config
cd sno-config

```

---

## Logical Configuration (install-config.yaml)

This file defines the cluster name, the base domain, and the security trust bundle.

```yaml
apiVersion: v1
baseDomain: <domain_name>
compute:
- name: worker
  replicas: 0
controlPlane:
  name: master
  replicas: 1
metadata:
  name: <cluster_name>
platform:
  none: {}
pullSecret: '<merged_pull_secret>'
sshKey: '<ssh_public_key>'
additionalTrustBundle: |
  -----BEGIN CERTIFICATE-----
  <registry_ca_contents>
  -----END CERTIFICATE-----

```

---

## Physical Configuration (agent-config.yaml)

This file maps the logical cluster to the physical hardware, including static networking and disk selection.

```yaml
apiVersion: v1
kind: AgentConfig
metadata:
  name: <cluster_name>
hosts:
  - hostname: <sno_hostname>
    interfaces:
      - name: <interface_name>
        macAddress: <mac_address>
    networkConfig:
      interfaces:
        - name: <interface_name>
          type: ethernet
          state: up
          ipv4:
            enabled: true
            address:
              - ip: <sno_node_ip>
                prefix-length: 24
      dns-resolver:
        config:
          server:
            - <dns_server_ip>
      routes:
        config:
          - destination: 0.0.0.0/0
            next-hop-address: <gateway_ip>
            next-hop-interface: <interface_name>

```

---

## Hardware & Path Precision

For a successful SNO boot in a disconnected environment, the following hardware details must be verified.

| Component | Requirement | Recommendation |
| --- | --- | --- |
| Interface Name | Must match the RHCOS kernel name (e.g., eno1, ens3) | Verify via 'ip link' on a Live ISO if unsure |
| MAC Address | Must match the physical NIC intended for PXE/Boot | Double-check against physical labels or BIOS |
| Installation Disk | The target drive for the RHCOS operating system | Use /dev/sda or /dev/nvme0n1 consistently |
| Disk Pathing | Persistence across reboots in varied hardware | Use /dev/disk/by-id/ for unambiguous identification |

---

## Architectural Justifications & Reference Notes

| Category | Technical Requirement Details | Documentation Source |
| --- | --- | --- |
| Interface Mapping | In Agent-based installs, if the interface name in networkConfig does not match the hardware, the static IP will not bind, causing the node to be unreachable. | [Agent-based Installer Guide](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_an_on-premise_cluster_with_the_agent-based_installer/index) |
| Disk Selection | The installer defaults to the first available disk. Forcing a specific disk path prevents accidental overwrites of existing data drives. | [SNO Preparing to Install](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_on_a_single_node/installing-sno-preparing-to-install-on-a-single-node) |
| replicas: 0 | In a Single Node OpenShift deployment, worker replicas must be set to 0 as the master node handles both roles. | [Installing SNO Overview](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_on_a_single_node/index) |
| additionalTrustBundle | This field is mandatory for disconnected installs. Without it, the node cannot verify the TLS certificate of the local Quay registry. | [Disconnected installation mirroring](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/disconnected_installation_mirroring/index) |

---
