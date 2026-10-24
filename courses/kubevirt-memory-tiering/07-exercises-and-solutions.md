# 07 — Check your understanding

Try the questions before reading the solution guidance.

## Questions

1. A fio read targets a VM page whose host backing is resident in DRAM. What path does the read take? How does the path change if the host backing was swapped out?
2. What are the two region roles in the case study? How many measurement working-set sizes are used?
3. Who touches the newly added bytes in a region, and what action makes them real page contents?
4. Why does the test use both fio latency and sysbench memory bandwidth?
5. A benchmark has 10,000 read observations. What does p99 mean?
6. If node swap use is 2 TiB, can that value alone tell you how many bytes of the measurement VMs' memory are swapped out?
7. Why is the report's Used-minus-RSS ratio only a proxy for swapped guest pages?
8. What evidence would you want before applying this result to a write-heavy database workload?

## Solution guidance

1. With resident host backing, the guest access proceeds using a host RAM page. If it was swapped out, the host must fault it back in from NVMe, adding delay before the guest can continue.
2. The roles are measurement regions, which fio reads, and pressure regions, which increase memory demand. There are three measurement sizes: 25%, 50%, and 75% of 92 GiB.
3. The in-guest region daemon writes/fills the newly extended tmpfs file bytes with pseudo-random data. The report does not provide the daemon's exact code path, so do not assume details beyond that description.
4. fio reports random-read latency and its tail. sysbench provides a sequential bandwidth cross-reference. Throughput and per-read latency are different properties.
5. p99 is the latency threshold at or below which about 99% of observed reads completed; the slowest 1% took longer.
6. No. The node-level counter is total node swap occupancy. It does not identify which process or VM owns every byte.
7. Used is a domain-view metric; RSS is the host QEMU process's resident set. Their difference is not a direct per-page host-swap counter, so it should be interpreted as an approximate placement signal.
8. At minimum: representative writes and reads, writeback behavior, realistic locality, burst patterns, service-level latency percentiles, adequate repetitions, multiple nodes, and telemetry that attributes host memory and swap behavior to the workload.
