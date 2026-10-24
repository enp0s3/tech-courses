# 06 — Evaluate the evidence

## Goal

State what this benchmark tested, what its working-set tiers contribute, and where its conclusions stop.

## Why 25%, 50%, and 75%?

The three fixed regions create small, middle, and large measurement working sets relative to each guest's 92 GiB RAM. They let the reader compare latency by working-set size while node pressure changes. The report relates the design to the “working set fits in RAM” principle.

The percentages are coarse experimental bins, not a standard or a guarantee about placement. They are percentages of **one guest's** RAM, not percentages of the worker's 1 TiB of DRAM. Actual residency depends on all active VM pages, total node pressure, and access history. The report does not derive why these exact cut points were chosen from a production workload distribution.

One VM per tier makes the comparison easy to follow but gives little statistical power. The report explicitly notes it cannot calculate confidence intervals from this design.

## What the results illustrate

At high pressure, the raw step table reports the 25% tier's p99 fio latency at 14.8 μs, compared with 146.4 μs for the 50% tier and 160.8 μs for the 75% tier. This illustrates why comparing working-set sizes is useful: the small-region test and the larger-region tests can have very different latency distributions under the same node pressure.

These measurements apply to this one synthetic random-read workload. They do not prove that a real service will have the same page locality, write behavior, latency, or failure behavior.

## Scope and limitations

The report describes one node and one run. It does not measure:

- pressure-VM fio latency;
- write-heavy or mixed I/O;
- realistic application access patterns;
- bursty target changes or transient latency during transitions;
- live migration or NUMA placement effects;
- cross-node repeatability or confidence intervals.

The benchmark also had no pre-warm for the first measurement sample. Its timing and sample-count descriptions do not fully agree, as discussed in lesson 04.

## A careful conclusion

A supportable conclusion is: under the tested setup, Linux host swap allowed substantial VM memory pressure while the smaller measurement working set retained much lower p99 latency than the larger tiers.

Do not turn that into a universal capacity promise. To evaluate a real service, reproduce its working-set size, read/write mix, locality, load bursts, guest configuration, and node topology; repeat across nodes and runs; and collect per-workload evidence.

## Check your understanding

Which claim is supported by the benchmark: “the small synthetic region had lower p99 latency than the larger regions in this run,” or “any production application will run without latency impact at 340% Active”? What additional evidence would the second claim need?

## Sources

- [KubeVirt architecture](https://kubevirt.io/user-guide/architecture/architecture/)
- [Linux memory-management concepts](https://docs.kernel.org/admin-guide/mm/concepts.html)
- [Linux tmpfs documentation](https://docs.kernel.org/filesystems/tmpfs.html)
