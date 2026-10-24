# References

These upstream sources support the technical explanations. Case-study figures in the lessons are labeled separately from documented platform behavior.

## KubeVirt

- [Architecture](https://kubevirt.io/user-guide/architecture/architecture/) — Kubernetes integration, scheduling, controllers, node daemons, and managed VM pods.
- [Metrics catalog](https://kubevirt.io/monitoring/metrics.html) — definitions for VM memory metrics, including Used and resident memory.
- [Component monitoring guide](https://kubevirt.io/user-guide/user_workloads/component_monitoring/) — examples of exposing and observing component metrics.

## Linux

- [Memory-management concepts](https://docs.kernel.org/admin-guide/mm/concepts.html) — page cache, anonymous memory, reclaim, and memory-pressure behavior.
- [tmpfs documentation](https://docs.kernel.org/filesystems/tmpfs.html) — tmpfs storage model and its ability to use swap.

## Benchmark tools

- [fio documentation](https://fio.readthedocs.io/en/latest/fio_doc.html) — I/O engines, memory mapping, direct/buffered I/O, and job options.
- [sysbench memory test source and tests](https://github.com/akopytov/sysbench/blob/master/tests/t/test_memory.t) — memory test options, access mode, operations, and reported throughput.
