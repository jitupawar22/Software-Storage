# Portworx Architecture Summary (from GitHub org review)

This repository folder contains **high-level architectural notes** derived from reviewing the **Portworx GitHub organization** and the ecosystem repositories commonly used to deploy Portworx on Kubernetes.

Sources referenced:
- Portworx GitHub org: `https://github.com/portworx`
- Key repos: `portworx/pxc`, `portworx/px-dev`, `portworx/helm`, `libopenstorage/stork`, `libopenstorage/operator`, `libopenstorage/openstorage`

---

### Core repository index (focus set)
- **Client / admin tooling**
  - **`portworx/pxc`**: Portworx client tooling; includes `pxctl`-style UX / kubectl-plugin approach for day-2 operations.
- **Orchestration layer (Stork)**
  - **`libopenstorage/stork`**: Storage Orchestration Runtime for Kubernetes; provides storage-aware scheduling and higher-level workflows for stateful apps.
- **Developer / local workflows**
  - **`portworx/px-dev`**: Developer-oriented packaging for running Portworx-style storage services in dev/test scenarios.
- **Cluster lifecycle / day-2 operations**
  - **`libopenstorage/operator`**: Kubernetes operator that manages Portworx deployment, upgrades, and configuration (and commonly orchestrates CSI enablement).
- **Deployment packaging**
  - **`portworx/helm`**: Helm charts to install Portworx and related components (including Stork/operator toggles).
- **Core storage/control APIs (OpenStorage lineage)**
  - **`libopenstorage/openstorage`**: OpenStorage APIs and related control-plane foundations used by Portworx integrations.

---

### System hierarchy (Control Plane vs Data Plane vs Orchestration)
- **Data Plane (IO path + on-node storage services)**
  - **Portworx runtime on worker nodes** (commonly deployed as a privileged **DaemonSet**): implements volume IO services, replication, snapshots, encryption, QoS, and media management.
  - **CSI Node plugin** (per-node): performs node-side publish/stage operations and presents volumes to Kubelet.
- **Control Plane (cluster state, provisioning decisions, lifecycle)**
  - **KVDB**: the shared cluster metadata/state store for Portworx (nodes, volumes, replication sets, policies, runtime state).
  - **CSI Controller plugin**: handles Create/Delete/Expand and attach/detach-style control loops by calling Portworx control APIs.
  - **Operator**: manages installation/upgrade/config drift and the desired state of Portworx + CSI + add-ons.
- **Orchestration Layer (Stork)**
  - **Stork**: Kubernetes extension layer for storage-aware scheduling and stateful workflows (data locality, migrations, DR hooks, policy-driven orchestration).

---

### Core components (roles in the ecosystem)
- **KVDB**
  - **Role**: distributed source of truth for Portworx cluster metadata and coordination.
  - **Why it matters**: enables consistent placement/failover/healing decisions across nodes independent of Kubernetes objects.
- **Portworx CSI**
  - **Role**: Kubernetes-standard interface translating PV lifecycle events to Portworx operations.
  - **Architecture**: split into **controller** (cluster-level) and **node** (host-level) components.
- **Stork**
  - **Role**: orchestrates storage-aware Kubernetes behaviors beyond base CSI (notably pod placement with data locality and app-centric workflows).

---

### Data flow (PVC → Portworx → disk)
- **1) PVC created**
  - User defines a `PersistentVolumeClaim` referencing a `StorageClass` (often including Portworx-specific parameters like replication, encryption, profiles).
- **2) Provisioning (control plane)**
  - CSI controller receives `CreateVolume` and calls Portworx control APIs.
  - Portworx consults **KVDB** for cluster state and selects placement/replica sets.
- **3) Volume materialization (data plane)**
  - Portworx runtime allocates on backing media and establishes replication as required.
  - Kubernetes binds a `PersistentVolume` (PV) to the PVC.
- **4) Scheduling**
  - Kubernetes schedules the consuming Pod; **Stork may influence placement** for data locality when enabled/used.
- **5) Node publish/mount**
  - CSI node plugin stages/publishes the volume and mounts it into the pod’s filesystem namespace.
- **6) Application IO**
  - Pod IO flows through the mounted device/filesystem to the Portworx runtime and onto the underlying disks (and to replicas if configured).

---

### Architecture patterns observed
- **Operator-based**
  - Operator manages lifecycle, upgrades, and desired state enforcement.
- **DaemonSet-based**
  - Portworx runtime on each eligible node; CSI node plugin is naturally per-node.
- **Deployment + CSI sidecar pattern**
  - CSI controller plugin commonly runs as a deployment and may include standard CSI sidecars (provisioner/attacher/resizer/snapshotter) depending on packaging/version.
- **Mix**
  - Portworx on Kubernetes is typically a **hybrid** of operator + daemonset + controller deployments.

---

### Integration points (cloud vs bare metal)
- **AWS / Azure / GCP**
  - **Backing media**: typically cloud block devices.
  - **Topology**: respects zone/region constraints via Kubernetes topology and StorageClass configuration.
  - **Ops**: operator/helm driven; node churn/autoscaling interacts with replica healing and membership.
- **Bare metal / on-prem**
  - **Backing media**: local devices (SSD/NVMe/HDD) and/or SAN-presented LUNs.
  - **Networking**: replication traffic uses cluster networking; failure domains are explicitly modeled (rack/row).
  - **Ops**: capacity growth and device management are handled through Portworx runtime + operator workflows.

---

### Next document
- See `Arch_Readme.md` for a **reference architecture** (boxes-and-arrows), Kubernetes primitive mapping, and the main control loops.
