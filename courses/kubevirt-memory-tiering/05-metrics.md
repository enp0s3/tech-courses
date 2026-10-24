# 05 — Metrics and their scope

## Goal

Identify where each memory metric comes from and avoid treating unlike counters as interchangeable.

## Used and RSS

The report uses these KubeVirt metrics in its derived ratio:

| Metric | Meaning in the report and upstream metric documentation |
|---|---|
| kubevirt_vmi_memory_used_bytes | Used memory as seen by the VM domain |
| kubevirt_vmi_memory_resident_bytes | Resident set size of the host process running the VM domain (QEMU) |

The resident metric is **host-process RSS**, not the guest operating system's own process RSS. The metric has VM labels, but the resident quantity describes the host process that runs the VM.

## Tiering ratio

The report defines:

    tiering_ratio =
      sum(Used bytes - resident bytes) / sum(Used bytes)

The numerator attempts to estimate how much domain-used memory is not represented in host QEMU RSS; dividing by Used turns it into a share. The report interprets that share as VM memory outside host RAM.

Treat this as a **proxy**, not an exact count of swapped guest pages. The two inputs describe different layers: domain-view Used and host-process RSS. RSS can include process memory beyond guest page contents, and the difference is not itself a per-page swap counter. The ratio is useful alongside the other measurements, but it cannot precisely attribute every host swap byte to guest memory.

## Host swap and host pressure

The report also calculates node-level swap use as:

    node_memory_SwapTotal_bytes - node_memory_SwapFree_bytes

This answers “how full is the node's swap area?” It is not a per-VM attribution and not an I/O rate. Swap-in and swap-out counters or rates answer how much swap traffic is occurring over time.

The node's Active memory is also host-wide. It describes the host kernel's memory state, not a per-VM working-set measurement.

## Measurement layers at a glance

    Guest/domain view: Used bytes, guest Active (if guest agent reports it)
    Host process view: resident RSS of the QEMU domain process
    Host node view: Active memory, swap capacity used, swap traffic
    fio view: completion latency for reads in the measurement guest

The report's pipeline collected KubeVirt and node metrics through Prometheus and Thanos at each step. Keep the measurement layer attached to each number when interpreting charts.

## Check your understanding

If the node reports 2 TiB of swap used, does that mean the benchmark VMs collectively have exactly 2 TiB of guest pages swapped out? Why or why not?

## Sources

- [KubeVirt metrics](https://kubevirt.io/monitoring/metrics.html)
- [KubeVirt component monitoring](https://kubevirt.io/user-guide/user_workloads/component_monitoring/)
