# NetApp: Architectural Deep Dives

This directory documents the **technical architecture** of widely deployed NetApp storage platforms. Each product has its own deep-dive README, while this file acts as an index.

---

## Products covered
* **ONTAP (AFF/FAS)**: `ONTAP_README.md`
    * Focus: WAFL layout, NVRAM/NVLOG durability, consistency points, RAID-DP/RAID-TEC, Snapshots/SnapMirror at a workflow level.
* **StorageGRID (Object Storage)**: `StorageGRID_README.md`
    * Focus: grid node roles/services, ILM policy engine, replication vs erasure coding, metadata store, ingest/retrieve flows.

---

## Reading order (suggested)
* Start with **ONTAP** if you want file/block semantics + crash-consistent write pipeline.
* Start with **StorageGRID** if you want S3 object semantics + ILM-driven placement and EC.

