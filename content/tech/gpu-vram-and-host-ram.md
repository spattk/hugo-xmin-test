---
title: GPU VRAM and Host RAM
date: 2026-09-08T04:59:00.000-07:00
author: Sitesh Pattanaik
---
understanding differences between gpu vram and host ram

### GPU VRAM

* otherwise known as HBM - high bandwidth memory
* sits directly on the GPU package, connected by a very short, very wide bus.
* on H100, that's 80GB of HBM3 at 3.35 TB/s.
* on B200, it's 192GB of HBM3E at 8 TB/s.

### Host RAM

* DDR5 is much larger -- 1-2 TB per server but far slower in bandwidth and latency
* slower as it's not on GPU, the GPU has to use PCIe  -- portable component interconnect -- to connect to the host RAM

#### practical consequence

* if the model + KV cache doesn't fit in VRAM, you either shard the model across multiple GPUs, quantize or offload to host RAM.
* offloading to host RAM is pretty slower because every access would go through the PCIe.

### PCIe Channels
- Peripheral Component Interconnect Express

- This is a general purpose high speed bus used to connect the CPU/host to add in ards like GPUs, NVMe SSDs, network cards, etc.

* PCIe Gen15 x16 gives roughly 64GB/s in each direction between CPU/Host Ram and GPU. Compare to that of 3-8TB/s of HBM bandwidth inside the GPU. This link is used for

  * Loading weights/data from host into VRAM initially.
  * GPU-GPU traffic when the NVLink isn't available.

### NVLink/ NVSwitch

This made possible multiple GPU to talk to each other and to behave as a single GPU.

* **NVLink**: a direct GPU to GPU interconnect that bypasses the CPU/PCIe entirely. H100(NVLink 4) gives ~900GB /s aggregate per GPU; B200 (NVLink5) roughly doubles that to 1.8TB/s.
* NVSwitch: a switch chip that connects all GPUs in a node so any GPU can talk to any other GPU at full NVLink bandwidth -- not just neighbour to neighbour. A DGX H100 node has 8 GPUs fully messed this way. NVIDIA's newer NVL72 rack-scale systems extend this switch to 72 GPUs.



#### Order

* Object Storage (S3) - 1-10GB/s
* NVMe SSD (local cache) - 5-7 GB/s per drive
* Host RAM (DDR5) - 200-400GB/s
* GPU VRAM (HBM3/ HBM3e) - 3000-8000 GB/s

#### practical consequence

* This is the exact reason cold start latencies is dominated by object stores and the steady state inference speed is dominated by HBM.
* Once weights are instated in VRAM, the GPU doesn't need to travel the hierarchy.



#### InfiniBand / RoCE

InfiniBand: The propritary technology of NVIDIA to do RDMA (remote direct memory access) -- access remote computer's RAM without involving remote computer's CPU.

RoCE - RDMA over Converged Ethernet - is a generic way to do RDMA.

note:

- the whole process of RDMA would require some form of CPU computation to translate the virtual address into physical address space. how does that work?
- that's the whole point of a memory registration phase with something called as RNIC (RDMA enabled NIC) which handles the address translation
- ordering wise, when a host computer is ready to use RDMA, it notifies the host CPU and blocks a certain part of virtual memory to be used for RDMA, and the tranlation sheet is handed over to RNIC for future translation making the host CPU free.


#### Why PCIe Exists?
- A CPU can't directly talk to all the peripheral devices using custom wiring -- there are too many kinds of devices.
- hence, computers standardize on a shared I/O bus on which device manufacturers can build against.
- this enables peripheral devices to read/write from host RAM without CPU. otherwise, all the essential cycles of CPU would be lost copying the data over.
- **Interrupts** - this is a way for the peripheral devices to notify CPU that it's completed the work.









