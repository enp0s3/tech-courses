# 03 — fio and sysbench

## Goal

Understand why the benchmark used two tools and why their results cannot be substituted for each other.

## fio: random-read latency

The case study ran fio inside each measurement guest with:

    fio --ioengine=mmap --rw=randread --direct=0

This is a partial command excerpt, not a complete reproducible command: the write-up does not show all job, file, block-size, or runtime arguments in this line. Elsewhere it describes random 4 KiB reads and reports p50, p99, and p99.9 latency.

- **randread** selects read locations in a random order.
- **mmap** accesses a mapped file through memory addresses. If a needed page is not resident, the access can fault and wait for the kernel to make it available.
- **direct=0** requests buffered rather than non-buffered I/O. The intent here is to exercise memory mapping and normal page handling, not to benchmark a raw NVMe data file.

The workload target is a tmpfs region in the guest. The host's NVMe swap is a possible backing tier for VM pages; fio is not issuing file reads directly to the host swap partition.

## sysbench: sequential memory bandwidth

The write-up says sysbench memory ran inside a guest as a cross-reference and reported sequential memory bandwidth in MiB/s. Bandwidth measures how much data a test moves per second. It does not answer how long an individual random read takes.

| Tool/result | Main question |
|---|---|
| fio p50/p99/p99.9 latency | How long do random reads take, including slow swap-related cases? |
| sysbench memory bandwidth | How much data per second does a sequential memory test move? |

The report does not specify the exact sysbench options, whether the operation was read or write, or which guest ran it. Avoid inventing those details.

## Why use both?

The measurements are complementary. A sequential bandwidth baseline can show that a guest's basic memory-throughput result remains plausible while fio exposes long-tail stalls from random page access. A high bandwidth result does not prove that random reads have low tail latency.

## Check your understanding

If sysbench reports high MiB/s, can you conclude that the 99th-percentile fio read has low latency? Explain.

## Sources

- [fio documentation: I/O engines and I/O type](https://fio.readthedocs.io/en/latest/fio_doc.html)
- [sysbench memory test options and output](https://github.com/akopytov/sysbench/blob/master/tests/t/test_memory.t)
