# 01 — Memory tiers and page reclaim

## Goal

Understand why a memory access can be fast in one case and much slower in another.

## A page moves through several views

Programs use **virtual addresses**. The guest operating system maps those addresses to guest physical pages. KVM/QEMU and the host kernel back guest memory with host memory pages. In this test, the host may keep those pages in DRAM or move reclaimable pages to its NVMe swap area.

The guest and host are separate operating-system layers. A page the guest considers useful can still have host backing that was swapped out. When the guest uses that page again, the host must restore its backing before the VM can continue.

## Reclaim and swap

Linux tracks recently used and less recently used pages. Under memory pressure, it reclaims pages it can free or move elsewhere. Swap is one destination for anonymous or otherwise swap-capable memory. A later access to a swapped-out page causes a **major page fault**: the process waits while the kernel reads the page back.

In simplified terms:

    access to resident page  -> RAM access -> continue
    access to swapped page   -> fault -> NVMe read -> page restored -> continue

Swap increases the amount of memory the node can support, but a swap-backed read takes far longer than a DRAM access. The test asks whether the host can keep the measurement workload's frequently accessed pages in DRAM while other VM pages occupy swap.

## Host swap versus guest swap

This course is about the **host's** swap device described in the case study. Do not infer that the guest operating systems also enabled or used swap; the case write-up does not establish that. The host's swap partition is a separate layer from each guest's filesystems.

## Why access frequency matters

A page touched recently is more likely to be reused soon. The kernel can favor keeping such pages resident. A cold page that is not touched for a while is a better reclaim candidate. This is a heuristic based on observed access, not knowledge of application intent.

The benchmark's measurement regions are repeatedly sampled by fio. Pressure-region pages are populated to consume memory, but the write-up does not fully document how often pressure daemons revisit already-filled pages afterward.

## Example

Suppose VM A repeatedly reads a 20 GiB region while VM B has filled a much larger region and no longer revisits much of it. As pressure rises, the host may keep more of A's active pages in DRAM and reclaim some of B's colder pages to swap. If A later touches one of its swapped pages, the access stalls for swap-in.

The example illustrates the intended tiering behavior. It does not guarantee Linux will choose exactly those pages; host page classification and overall pressure determine reclaim.

## Check your understanding

Why does swap usage matter to latency only when a workload later touches a page that is no longer resident?

## Sources

- [Linux memory-management concepts](https://docs.kernel.org/admin-guide/mm/concepts.html)
- [Linux tmpfs documentation](https://docs.kernel.org/filesystems/tmpfs.html)
