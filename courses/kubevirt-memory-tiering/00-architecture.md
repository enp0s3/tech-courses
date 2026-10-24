# 00 — Setup architecture and data flow

## Goal

Build a mental model of where the VMs run, where their memory lives, and which process creates pressure or measures latency.

## KubeVirt layers

KubeVirt lets Kubernetes manage virtual machines alongside other workloads. Kubernetes schedules the VM instance onto a worker node. KubeVirt's node-side components launch and manage the VM; a VM's QEMU process runs in a KubeVirt-managed pod and uses KVM for hardware-assisted virtualization.

For this benchmark, the important boundary is between:

- **Inside a guest:** a small daemon and a tmpfs file make guest memory use.
- **On the worker host:** Linux backs the VM process with physical RAM and can reclaim host pages to the node's NVMe swap device.
- **In the measurement path:** fio runs in selected guests and reads their test regions.
- **In the telemetry path:** cluster monitoring collects KubeVirt VM metrics and node metrics.

KubeVirt's upstream architecture guide describes how its VM APIs, controllers, node daemons, and Kubernetes scheduling fit together. The diagram below focuses on the components involved in this experiment, not every component in a full cluster.

## Topology

    [Benchmark coordinator] -- target sizes --> [Region daemons in guests]

    +---------------------- One KubeVirt worker ------------------------+
    |                                                                   |
    |  VM guests                                                        |
    |    Measurement VMs x3                    Pressure VMs x9           |
    |    +---------------------+               +---------------------+   |
    |    | daemon -> tmpfs file|               | daemon -> growing   |   |
    |    | fio --random reads->|               | and filled tmpfs    |   |
    |    | fixed region        |               | regions             |   |
    |    +----------+----------+               +----------+----------+   |
    |               | guest pages backed by host VM processes |          |
    |               +----------------------+------------------+          |
    |                                      v                              |
    |  Linux host kernel:      +------------------+                       |
    |                          | DRAM             |                       |
    |                          | fast tier        |                       |
    |                          +--------+---------+                       |
    |                                   | reclaim / swap-in               |
    |                                   v                                 |
    |                          +------------------+                       |
    |                          | NVMe swap        |                       |
    |                          | slow tier        |                       |
    |                          +------------------+                       |
    +-------------------------------------------------------------------+

    [KubeVirt + node metrics] --> [Monitoring / query pipeline]

The case study used one worker with 1 TiB of DRAM and a 2.9 TiB NVMe swap partition. Those values describe that run; they are not KubeVirt requirements.

## Two paths through the setup

### Control path

1. The coordinator sends a target byte count for a benchmark step.
2. A daemon inside each guest adjusts its tmpfs region to the target size.
3. On growth, the daemon fills the newly added bytes with pseudo-random data.
4. The pressure VMs increase memory demand; the measurement VMs keep their configured region sizes.

### Memory and measurement path

1. A guest accesses its tmpfs data. The guest sees guest memory; the host sees pages backing the VM process.
2. When host RAM is under pressure, the Linux host may reclaim pages and move some backing pages to NVMe swap.
3. fio randomly reads the measurement VM's region. A resident page is available from RAM; if its host backing page was swapped out, access requires a host page fault and swap-in.
4. Monitoring records VM and node metrics so the read latency can be compared with memory pressure and swap use.

The worker's NVMe swap is a host-level memory tier. It is not the VM's tmpfs file and is not a data disk fio reads directly.

## What the case study contains

- 3 measurement VMs, each with 92 GiB guest RAM and a fixed region at 25%, 50%, or 75% of that amount.
- 9 pressure VMs, each with 400 GiB guest RAM and a region designed to grow to 370 GiB.
- One tmpfs region file per VM, maintained by the in-guest daemon.
- fio random reads only; the report does not test write-heavy application I/O, live migration, or failure recovery.

## Check your understanding

Without looking ahead, trace what happens when fio reads a measurement page that has been swapped out by the host. Then check the solution in lesson 07.

## Sources

- [KubeVirt architecture](https://kubevirt.io/user-guide/architecture/architecture/)
- [Linux memory-management concepts](https://docs.kernel.org/admin-guide/mm/concepts.html)
