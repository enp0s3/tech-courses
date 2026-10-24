# KubeVirt Memory Tiering: Onboarding Course

## Audience

Engineers who know the basics of Linux, containers, Kubernetes, and virtual machines, and want to understand how host RAM and NVMe swap behave when KubeVirt workloads overcommit memory.

## Prerequisites

- Familiarity with Linux processes, virtual memory, and files.
- Basic Kubernetes concepts: pods, nodes, and controllers.
- No cluster or benchmark environment is needed. The case-study values are supplied in the lessons.

## Learning outcomes

By the end, you should be able to:

1. Trace a guest memory access through KubeVirt, QEMU/KVM, host RAM, and host swap.
2. Explain the architecture and workload roles in a memory-tiering benchmark.
3. Distinguish allocating a region, touching its pages, and reading them later.
4. Explain what fio latency and sysbench bandwidth results do and do not show.
5. Read the benchmark's Used, RSS, node swap, and tiering-ratio metrics without confusing their scopes.
6. Separate measured findings from assumptions and limits.

## Why this matters

Memory overcommit can let a cluster run more guest memory than fits in physical RAM. The host then needs a way to reclaim pages and may place some on swap. Knowing which workload pages stay fast, which reads incur page faults, and what the metrics actually represent helps operators assess this design without mistaking a synthetic benchmark for a production guarantee.

## Recommended learning path

Read the lessons in order. Lessons 00–02 explain the system and its memory behavior; lessons 03–05 explain the measurements; lesson 06 evaluates the evidence. Finish with the exercises and glossary.

| Lesson | Topic | Approx. time |
|---|---|---:|
| [00 — Setup architecture and data flow](00-architecture.md) | KubeVirt components, worker node, VM groups, and telemetry | 10 min |
| [01 — Memory tiers and page reclaim](01-memory-tiers.md) | Guest memory, host RAM, swap, and page faults | 10 min |
| [02 — Working sets and regions](02-working-sets-and-regions.md) | tmpfs, touching pages, and pressure versus measurement VMs | 12 min |
| [03 — fio and sysbench](03-benchmark-tools.md) | Random-read latency versus sequential bandwidth | 12 min |
| [04 — Latency and test timing](04-latency-and-timing.md) | Percentiles, warm-up, ramp, and settle | 10 min |
| [05 — Metrics and their scope](05-metrics.md) | Used, QEMU RSS, host swap, and the derived ratio | 12 min |
| [06 — Interpreting the evidence](06-evaluating-results.md) | What the design supports and where it stops | 12 min |
| [07 — Check your understanding](07-exercises-and-solutions.md) | Exercises with solution guidance | 10 min |

Estimated total: about 90 minutes.

## Course conventions

- **Case-study fact** means the benchmark write-up explicitly reports it.
- **Documented behavior** means an upstream Linux, KubeVirt, fio, or sysbench source documents it.
- **Interpretation** means a reasoned explanation that connects those facts.
- **Unknown** means the write-up does not provide enough detail to verify it.

The course focuses on the open-source KubeVirt architecture and the reported setup.

## References

See [references.md](references.md) for upstream documentation and project sources.
