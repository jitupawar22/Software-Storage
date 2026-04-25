# Portworx Architecture & Design

Portworx is a software-defined storage platform designed specifically for containers and Kubernetes. It provides a distributed storage layer that abstracts physical storage (cloud drives or on-prem disks) into logical pools for stateful applications.

---

## 1. System Overview
* **Target Use Case:** Cloud-Native, Kubernetes-stateful sets (Databases, CI/CD).
* **Deployment Model:** Software-defined (Runs as a DaemonSet in Kubernetes).
* **Storage Types:** Block and File (via Shared Volumes).

## 2. Core Architecture & Components
* **Control Plane:** Distributed and headless. Every node runs a Portworx container that participates in the control plane via a gossip protocol (or KV store like etcd).
* **Data Plane:** The **STORK** (Storage Orchestrator for Kubernetes) scheduler and the **PX-Enterprise** storage engine manage local disk I/O and cross-node replication.
* **Key Services:**
    * **PX-Store:** The virtualization layer for local storage.
    * **PX-Central:** The management and monitoring dashboard.

## 3. Data Path & Write/Read Flow
* **Write Path:** Portworx intercepts writes at the block level. Data is written to the local node and synchronously replicated to "n" replica nodes (defined by the StorageClass) before acknowledging the write to the application.
* **Read Path:** Portworx prioritizes "Data Locality." If a pod is running on a node that holds a replica of the data, it reads locally. If not, it fetches data over the 10GbE/40GbE fabric.
* **Consistency Model:** Strongly consistent (Synchronous replication).

## 4. Scalability & Performance
* **Scaling Mechanism:** Scale-out. Adding a new Kubernetes node with disk automatically increases the cluster capacity.
* **Performance Tiers:** Supports storage profiles (e.g., `high`, `medium`, `low`) mapped to different drive types (NVMe vs. HDD).
* **Bottlenecks:** Network latency between nodes can impact synchronous write performance.

## 5. Resiliency & Data Integrity
* **Data Protection:** Replication-based (Replica 2 or Replica 3). 
* **Self-Healing:** If a node fails, Portworx informs Kubernetes to reschedule the pod on a node where a data replica exists.
* **Data Integrity:** Periodic background "scrubbing" to detect and repair bit rot.

## 6. Ecosystem Integration
* **Supported Protocols:** CSI (Container Storage Interface), NFS (for shared volumes).
* **Cloud Mobility:** Supports AWS (EBS/EFS), Azure (Managed Disks), and GCP (Persistent Disks).

---

### Reference Links
* [Portworx Official Documentation](https://docs.portworx.com/)
* [Portworx Architecture Whitepaper](https://portworx.com/resource/portworx-architecture-whitepaper/)
