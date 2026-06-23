# Step 5: Virtualization Activation & Day 2 Enablement

Created Date: June 23, 2026
Status: Post-Install Validation

Once the core multi-node platform is fully operational, the final phase transitions to unsealing your local software catalogs, deploying the hypervisor engines, and configuring node network policies. In an air-gapped environment, these steps must be executed systematically to ensure that the virtualization operators pull exclusively from your local registry and that physical interfaces are correctly bridged and exposed for production virtual machine routing.

---

## Post-Installation Verification Checklist

Before activating the virtualization tier, you must execute these cluster validation gates to ensure the underlying platform is stable and that image requests are successfully routing to your offline warehouse registry:

| Operational Core | Required Validation Metric | Inspection Verification Command |
| --- | --- | --- |
| Compute Pool Nodes | All Master and Worker nodes register an active `Ready` status. | `oc get nodes` |
| Platform Control Plane | All core ClusterOperators return `Available: True` and `Degraded: False`. | `oc get co` |
| Catalog Synchronization | Local OperatorHub catalog sources register an active `READY` status. | `oc get catalogsource -n openshift-marketplace` |

---

## Sequential Operator Initialization (Step 5.1)

Navigate to the OperatorHub via the cluster web console or utilize the command line interface to subscribe to the required operators. 

> **NOTE:** Do not copy and paste these configuration blocks directly. They are abstract structural reference examples of the custom resources required to initialize the environment. You must adjust metadata fields, names, interfaces, and parameters to align with your specific cluster naming conventions and target namespaces.

### 1. Initialize the Core HyperConverged Cluster Instance (with Migration Isolation)
Applying this manifest commands the virtualization operator to unpack and deploy the foundational hypervisor components across all eligible worker nodes, while explicitly dedicating a secondary network interface for high-throughput live-migration traffic.

```yaml
apiVersion: hco.kubevirt.io/v1beta1
kind: HyperConverged
metadata:
  name: kubevirt-hyperconverged
  namespace: openshift-cnv
spec:
  liveMigrationConfig:
    network: vlan-100-vm-network
```

### 2. Configure Node Network Bridging Policies (`NodeNetworkConfigurationPolicy`)
This custom resource commands the NMState engine to bind physical interfaces on your compute nodes to a standard layer-2 network bridge to handle external, traditional virtual machine traffic routing.

```yaml
apiVersion: nmstate.io/v1
kind: NodeNetworkConfigurationPolicy
metadata:
  name: vm-vlan-bridge-policy
spec:
  nodeSelector:
    node-role.kubernetes.io/worker: ""
  desiredState:
    interfaces:
      - name: br-vlan
        type: linux-bridge
        state: up
        bridge:
          options:
            stp:
              enabled: false
          port:
            - name: eth1
    routes:
      config: []
```

### 3. Expose the Host Bridge to Virtual Machine Namespaces (`NetworkAttachmentDefinition`)
This configuration tells the Multus network engine to map the physical host bridge (`br-vlan`) into a selectable network asset inside the target virtual machine namespace so that virtual NICs can actually bind to it.

```yaml
apiVersion: "k8s.cni.cncf.io/v1"
kind: NetworkAttachmentDefinition
metadata:
  name: vlan-100-vm-network
  namespace: target-vm-namespace
spec:
  config: '{
      "cniVersion": "0.3.1",
      "name": "vlan-100-vm-network",
      "type": "cnv-bridge",
      "bridge": "br-vlan",
      "macspoofchk": true
    }'
```

---

## Storage Realignment & Template Ingestion (Step 5.2)

Because the default templates are configured to dynamically stream operating system images from public cloud endpoints over the internet, you must manually point the Containerized Data Importer (CDI) to your offline image volumes.

* **Shared Storage Class Mapping:** Ensure your underlying Container Storage Interface (CSI) drivers or enterprise storage arrays are presenting block-mode storage classes supporting ReadWriteMany (RWX) access modes. This foundation is mandatory to support non-disruptive, multi-node live migrations.
* **Offline Boot Source Ingestion:** Download required guest operating system ISOs or QCOW2 cloud images on your connected staging bastion, transport them across the air-gap perimeter, and upload them directly into the cluster's internal storage repository using the `virtctl` utility. Update your deployment templates to reference these local storage locations exclusively.

---

## Virtual Machine Deployment Verification (Step 5.3)

Validate the operational status of the hypervisor tier by launching a test virtual machine workload natively offline.

> **NOTE:** Do not copy and paste these commands directly. This block represents a mock reference diagnostic sequence and must be executed using your explicit workload names and target namespaces.

```bash
# 1. Example syntax for verifying that the hypervisor components are deployed and running
oc get csv -n openshift-cnv

# 2. Reference workflow for auditing the state of the physical network bridge on your nodes
oc get nncp vm-vlan-bridge-policy -o yaml

# 3. Structural verification checking that the test virtual machine has entered an active Running state
oc get vmi -n target-vm-namespace
```

---

## Architectural Justifications & Reference Notes

| Category | Technical Requirement Details | Documentation Source |
| --- | --- | --- |
| Catalog Verification Gates | Monitoring your offline `CatalogSource` manifests prevents cascading installation blocks; if a catalog source remains stuck in a `Pending` state, it typically highlights an unverified or missing registry CA trust file. | [Operator Lifecycle Maintenance Guide](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/openshift_virtualization/installing-virt) |
| Bridge Network Segregation | Utilizing a dedicated interface (such as `eth1`) for your `NodeNetworkConfigurationPolicy` isolates high-throughput VM data traffic from the latency-sensitive control plane lines of the underlying OpenShift cluster nodes. | [Advanced Interface Management via NMState](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/networking/index#virt-networking-with-kubernetes-nmstate) |
| Multus Logical Bridge Mapping | The `NetworkAttachmentDefinition` (NAD) acts as the mandatory link between Kubernetes object namespaces and the node's bare-metal kernel bridge. Without an active NAD, the virtual machine engine cannot provision virtual network interface cards (vNICs). | [Exposing Node Bridges via Multus](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/networking/index#virt-networking-with-kubernetes-nmstate) |
| Live Migration Isolation | Forcing memory-state serializations onto a dedicated secondary network lane (`vlan-100-vm-network`) prevents high-frequency VM migrations from inducing artificial latency spikes on the cluster's internal `etcd` and management control pathways. | [Configuring Virtual Machine Live Migration](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/openshift_virtualization/installing-virt) |
| Multi-Node Live Migration | Live-migration engines require identical underlying block storage presentation across all worker nodes; failing to enforce ReadWriteMany (RWX) access modes blocks the hypervisor from serializing memory states during a host failover event. | [OpenShift Virtualization Storage Requirements](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/openshift_virtualization/installing-virt) |
| Boot Source Disconnection | Overriding default cloud templates with local data volumes avoids permanent deployment hangs; without an outbound internet route, public cloud image streaming calls will time out indefinitely. | [Managing Virtual Machine Boot Sources](https://docs.redhat.com/en/documentation/openshift_container_platform/4.16/html/openshift_virtualization/installing-virt) |
