# 02 — Working sets and regions

## Goal

Connect the abstract idea of a working set to the concrete tmpfs files used by the benchmark.

## Working set

A workload's **working set** is the data it is actively using over a relevant time window. A small working set may fit in fast memory even when the workload's total address space is large. If the active data does not fit, some accesses may have to wait for data to return from a slower tier.

For this synthetic benchmark, the region is the fio access domain: fio chooses random 4 KiB reads from the measurement VM's region. It is not a realistic database or application access pattern, and a random-read sample is not necessarily a full scan of every byte.

## The region is a tmpfs file

Inside each guest, a small region daemon maintains a tmpfs file such as /dev/shm/region.bin. The coordinator writes a target size; the daemon extends or truncates the file to match. On growth, it fills the new bytes with pseudo-random data.

That distinction matters:

- **Setting a file's length** describes the file's logical size.
- **Writing the new bytes** touches pages and makes their contents real.
- **Reading later** accesses those pages again and may cause a fault if backing is no longer resident.

Linux tmpfs stores file data in virtual memory and, unless configured otherwise, allows tmpfs pages to be swapped under memory pressure. The benchmark uses this property to create reclaimable memory without using a normal disk file as the workload data.

## Two region roles, three measurement sizes

| VM group | Count | Region behavior | Purpose |
|---|---:|---|---|
| Measurement | 3 | Fixed regions sized to 25%, 50%, or 75% of 92 GiB guest RAM: about 23, 46, or 69 GiB | fio reads these regions to compare latency by working-set size |
| Pressure | 9 | Regions grow together, with up to 370 GiB per 400 GiB guest | Filled pages increase aggregate memory demand |

All VMs use the same kind of tmpfs file. The two roles are not two different storage technologies. The NVMe swap partition is a separate host-level destination chosen by the host kernel.

The measurement regions stay fixed while the pressure VMs drive the node through the staircase. That holds the measurement workload sizes steady as overall pressure changes. The three percentages are broad comparison points, not a standard or a formula that predicts the exact share swapped.

## Allocation, touch, reclaim, and read

    coordinator raises target
        -> daemon grows and fills pressure tmpfs region
        -> guest pages consume memory backed by host VM pages
        -> host pressure makes some pages reclaim candidates
        -> host may swap selected pages to NVMe
        -> fio later reads measurement region
        -> resident page is quick; swapped backing must be paged in

The region daemon performs the write that fills newly added bytes. fio later performs reads against measurement regions. The write-up does not specify how frequently pressure daemons touch old pages after filling them, so do not assume every pressure page remains hot or becomes cold at a fixed time.

## Check your understanding

If a tmpfs region grows by 5 GiB and the daemon fills the new data, what creates actual memory pressure: the size change alone, or the subsequent writes? Why can a later fio read still be slow?

## Sources

- [Linux tmpfs documentation](https://docs.kernel.org/filesystems/tmpfs.html)
- [Linux memory-management concepts](https://docs.kernel.org/admin-guide/mm/concepts.html)
