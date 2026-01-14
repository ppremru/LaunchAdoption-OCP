# Step 4: Crafting SNO Configurations (Disconnected)

Created Date: January 14, 2026

Status: Configuration Phase

This document guides the administrator through the creation of the two core YAML files required by the Agent-based Installer to deploy a Single Node OpenShift cluster. These files define the network, identity, and hardware specifics for the target node.

## Configuration Overview

| Task | Description |
| --- | --- |
| **install-config.yaml** | Defines the cluster's base identity, including the cluster name, base domain, networking CIDRs, and the merged pull secret. |
| **agent-config.yaml** | Defines the physical hardware details, such as static networking, disk selection, and the host's BMC (Baseboard Management Controller) information. |
| **Automation Preparation** | Ensuring all variables (MAC addresses, IPs, and FQDNs) are gathered before generating the ISO. |

### SNO Configuration Reference Script

> **Note**: These files represent a **baseline functional guide**. Administrators must replace placeholders (marked with `< >`) with values specific to their local environment.

```yaml
# 1. Cluster Definition
# File: install-config.yaml
apiVersion: v1
baseDomain: <your_base_domain>
metadata:
  name: <cluster_name>
compute:
- name: worker
  replicas: 0 # SNO requires 0 worker replicas
controlPlane:
  name: master
  replicas: 1
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  networkType: OVNKubernetes
  serviceNetwork:
  - 172.30.0.0/16
platform:
  none: {}
fips: true # Enabled for hardened solution requirements
pullSecret: '<your_merged_pull_secret_content>'
sshKey: '<your_public_ssh_key_content>'

```

```yaml
# 2. Hardware and Network Definition
# File: agent-config.yaml
apiVersion: v1
kind: AgentConfig
metadata:
  name: <cluster_name>
hosts:
  - hostname: <node_hostname>
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
              - ip: <node_ip>
                prefixLength: <prefix_length>
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

## Technical Process & Justifications

| Category | Justification |
| --- | --- |
| **1. Replicas: 0** | In a Single Node OpenShift deployment, the master node handles both control plane and worker workloads; setting worker replicas to 0 is mandatory. |
| **2. NMState Config** | The `networkConfig` section uses NMState syntax to provide a declarative way to configure static IPs, which is required when DHCP is unavailable in the environment. |
| **3. SSH Key** | Providing a public SSH key during installation is the only way to gain "backdoor" access to the node for troubleshooting during the initial boot phase. |

---

## Architectural Justifications & Reference Notes

| Category | Technical Requirement Details | Documentation Source |
| --- | --- | --- |
| **Static IP Consistency** | The IP address assigned in `agent-config.yaml` must match the IP configured in your local DNS records (api, api-int, *.apps). | [Installing on a single node](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_on_a_single_node/index) |
| **Disk Selection** | By default, the installer picks the first available disk. If the node has multiple disks, use the `rootDeviceHints` in `agent-config.yaml` to pin the installation to the correct 120GB+ drive. | [Agent-based Installer Guide](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_an_on-premise_cluster_with_the_agent-based_installer/index) |
| **FIPS Compliance** | Enabling `fips: true` in `install-config.yaml` ensures the cluster utilizes FIPS-validated cryptographic modules from the moment of first boot. | [SNO Preparing to Install](https://www.google.com/search?q=https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_on_a_single_node/installing-sno-preparing-to-install-on-a-single-node) |

---

**Next Step:** With your YAML configurations finalized, would you like to move on to **05_iso_generation.md** to generate the final bootable ISO and begin the cluster verification?