# Dell: Architectural Deep Dives

Dell has multiple widely deployed storage product lines. This directory keeps **one architecture README per product**, and uses this file as an index.

---

## Products covered
* **PowerStore (unified block/file, NVMe-based array)**: `PowerStore_README.md`
    * Focus: active/active dual-node design, persistent write caching (NVMe NVRAM), Dynamic Resiliency Engine (DRE), data reduction pipeline, replication/metro volumes.
* **PowerScale (Isilon OneFS scale-out NAS)**: `PowerScale_README.md`
    * Focus: OneFS distributed filesystem, write “captain”/participant model, 2PC safe writes, FlexProtect erasure coding, SmartConnect, rebuild semantics.

