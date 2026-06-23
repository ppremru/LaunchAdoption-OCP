# Step 4: Infrastructure Configuration & Media Compilation

Created Date: June 23, 2026
Status: Manifest Definition

Deploying a highly available, multi-node cluster offline requires parallel management of your logical cluster definitions and physical node network configurations. The Agent-Based Installer consumes these structured assets to build a localized, bootable image file that automates node discovery and deployment natively offline.

---

## Pre-Flight Environment Requirements Checklist

Before generating the boot media, you must verify that the following infrastructure gates are active and correctly configured inside your air-gapped environment:

| Infrastructure Pool | Verification Gate Checkpoint | Verification Method |
| --- | --- | --- |
| Offline DNS | `api.<cluster_name>.<base_domain>` resolves to the external API load balancer or VIP. | `nslookup api.ocp-virt.domain.local` |
| Offline DNS | `*.apps.<cluster_name>.<base_domain>` resolves to the ingress router load balancer or VIP. | `nslookup console-openshift-console.apps.ocp-virt.domain.local` |
| Control Hardware | Three (3) distinct physical servers are allocated for the control plane. | Hardware Audit: Verify minimum 4 vCPUs and 16 GB RAM per host. |
| Compute Hardware | Multiple physical servers are allocated for the worker pool with active hypervisor extensions. | BIOS/UEFI Audit: Confirm Intel VT-x or AMD-V is set to Enabled. |
| Node OS Storage | Each physical server possesses a dedicated boot drive (**120 GB–200 GB+** SSD/NVMe RAID-1). | Hardware Audit: Ensure the platform OS disk is isolated from VM data drives. |
| Local VM Storage | Disks intended for local VM data storage (ODF/LVMS) are physically present in the server slots. | Storage Controller Audit: Verify these target storage drives are completely **raw and unpartitioned**. |
| External VM Storage | SAN/NAS storage networks, target IQNs, and dedicated VLANs are active on the network switch. | Network Fabric Audit: Cross-reference storage target ports with your physical switch configurations. |

---

## Workspace Setup & Blueprint Definition

Create a clean, dedicated orchestration workspace directory on your offline staging host to manage your configuration assets.

```bash
mkdir secure-cluster-config
cd secure-cluster-config
```

### 1. Logical Configuration Blueprint (`install-config.yaml`)
This configuration file outlines the baseline cluster profile, control/worker pool sizing, and the local container registry trust anchor.

> **NOTE:** Do not copy and paste this configuration file directly. This block is an abstract reference layout provided strictly as a structural guide. You must manually replace the placeholder domain parameters, network CIDRs, and root CA strings to match your target environment.

```yaml
apiVersion: v1
baseDomain: domain.local
metadata:
  name: ocp-virt-cluster
controlPlane:
  architecture: amd64
  hyperthreading: Enabled
  name: master
  replicas: 3
compute:
  - architecture: amd64
    hyperthreading: Enabled
    name: worker
    replicas: 3
networking:
  clusterNetwork:
    - cidr: 10.128.0.0/14
      hostPrefix: 23
  networkType: OVNKubernetes
  serviceNetwork:
    - 172.30.0.0/16
platform:
  baremetal:
    apiVIPs:
      - 192.168.10.10
    ingressVIPs:
      - 192.168.10.11
additionalTrustBundle: |
  -----BEGIN CERTIFICATE-----
  <INSERT_YOUR_INTERNAL_REGISTRY_ROOT_CA_HEX_DATA_HERE>
  -----END CERTIFICATE-----
pullSecret: '{"auths":{"<internal_registry_fqdn>:8443":{"auth":"xxxxxxxx"}}}'
```

### 2. Physical Host Network Blueprint (`agent-config.yaml`)
This secondary configuration asset maps static network configurations, gateway paths, storage network interfaces, and physical interfaces directly to individual bare-metal server MAC addresses before cluster provisioning begins.

> **NOTE:** Do not copy and paste this configuration file directly. This model shows the structural layout for mapping multiple nodes statically, including dedicated storage interfaces. You must expand this block to define all control plane and compute worker hosts in your environment.

```yaml
apiVersion: v1alpha1
kind: AgentConfig
metadata:
  name: ocp-virt-cluster
rendezvousIP: 192.168.10.20
hosts:
  - hostname: master-01.domain.local
    role: master
    interfaces:
      - name: eth0
        macAddress: 00:1a:4a:16:01:aa
    networkConfig:
      interfaces:
        - name: eth0
          type: ethernet
          state: up
          ipv4:
            enabled: true
            address:
              - ip: 192.168.10.20
                prefixLength: 24
      dns-resolver:
        config:
          server:
            - 192.168.10.5
      routes:
        config:
          - destination: 0.0.0.0/0
            next-hop-interface: eth0
            next-hop-address: 192.168.10.1
            table-id: 254
  - hostname: worker-01.domain.local
    role: worker
    interfaces:
      - name: eth0
        macAddress: 00:1a:4a:16:02:bb
      - name: eth1
        macAddress: 00:1a:4a:16:02:cc
    networkConfig:
      interfaces:
        - name: eth0
          type: ethernet
          state: up
          ipv4:
            enabled: true
            address:
              - ip: 192.168.10.30
                prefixLength: 24
        # Dedicated network path for external SAN/NAS storage fabric or VM guest bridging
        - name: eth1
          type: ethernet
          state: up
          ipv4:
            enabled: true
            address:
              - ip: 192.168.20.30
                prefixLength: 24
      dns-resolver:
        config:
          server:
            - 192.168.10.5
      routes:
        config:
          - destination: 0.0.0.0/0
            next-hop-interface: eth0
            next-hop-address: 192.168.10.1
            table-id: 254
```

---

## Compiling Bootable Installation Media

Before compiling the bootable artifact, copy the `ImageDigestMirrorSet` (IDMS) translation manifests generated by the mirroring tool into your active configuration workspace directory. This step ensures image requests route correctly to your offline warehouse registry.

> **NOTE:** Do not copy and paste these commands directly. This block is an abstract reference sample showing the asset compilation sequence and must be executed within your local configuration directory.

```bash
# 1. Back up your foundational configuration blueprints to prevent installer asset consumption
mkdir -p ../blueprint-backups
cp install-config.yaml agent-config.yaml ../blueprint-backups/

# 2. Trigger the installer engine to compile the unified configuration files into a bootable image
openshift-install agent create image

# 3. Confirm that the self-contained installation media compiled successfully
ls -lh agent.iso
```

---

## Architectural Justifications & Reference Notes

| Category | Technical Requirement Details | Documentation Source |
| --- | --- | --- |
| Static Network Architecture | Utilizing the Agent installer configuration topology avoids provisioning and maintaining separate, complex DHCP or PXE server environments inside restricted network zones. | [Agent-Based Installer Customization](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_a_cluster_using_the_agent-based_installer/) |
| Rendezvous IP Selection | The designated `rendezvousIP` defines which node acts as the ephemeral control coordinator during initial provisioning. All cluster hosts monitor this IP to synchronize their state offline. | [Agent installer Parameter Reference](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_a_cluster_using_the_agent-based_installer/) |
| Manifest Preservation | The `openshift-install` binary consumes configuration manifests during execution. Maintaining standalone configuration backups prevents losing your baseline work if you need to adjust parameters and rebuild the ISO. | [Generating Clusters via Agent Media](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_a_cluster_using_the_agent-based_installer/) |
| Upfront Storage Isolation | Forcing local VM data drives to remain completely raw and unpartitioned upfront allows Day 2 storage controllers (ODF/LVMS) to dynamically claim, provision, and cluster the disks without experiencing block device lock errors. | [OpenShift Virtualization Storage Planning](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/openshift_virtualization/installing-virt) |
| Core Performance Protection | Separating the platform operating system disks from the heavy, high-frequency random I/O of your virtual machine workloads ensures the underlying `etcd` database cluster maintains low write latencies, avoiding severe cluster degradation. | [Bare-Metal Cluster Infrastructure Requirements](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/installing_on_bare_metal/index) |
