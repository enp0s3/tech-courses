# 04 — Latency, percentiles, and timing

## Goal

Read fio's tail-latency measures and understand when benchmark timing can bias them.

## Percentiles

Latency is the elapsed time for an operation to complete. To calculate a percentile, sort the measured read times from fastest to slowest.

- **p50** is the median: half of the reads completed at or below that time.
- **p99** is the point at or below which 99% of reads completed; the slowest 1% took longer.
- **p99.9** focuses on the slowest 0.1%.

For 10,000 reads, p99 is around the 9,900th sorted observation. It does not mean a 99% slowdown. Percentiles describe the distribution of individual read times.

**Tail latency** means the slow end of the distribution, often p99 or p99.9. It matters because a small number of long waits can make an interactive or latency-sensitive workload feel stalled even when its average is acceptable.

## Warm-up and first-touch effects

A **pre-warm** runs the relevant access pattern before collecting timed samples. That first pass can fault pages in and make them recently used. The write-up says the measurement regions were not pre-warmed; it warns that the first fio sample in the first step may therefore include cold-cache or first-touch effects.

Warm-up is about the fio measurement regions and their access history. It is not the same operation as filling the pressure VM regions.

## Ramp and settle

The report describes a 180-second ramp before step measurements. In context, this is the period for raising pressure toward a step target by growing/filling pressure regions. The report does not provide the ramp algorithm or establish that it was perfectly linear.

After a target change, it describes a 120-second settle period, intended to give the memory system time to react before measuring. The benchmark then runs samples. A settle period does not guarantee production steady state.

## Timing details to treat carefully

The report gives two timing descriptions:

- Methodology says 20 minutes per step.
- Limitations describe a 180-second ramp, then 120 seconds of settling plus three 60-second benchmark samples, or about six minutes of measurement per step.

It does not reconcile these figures. The cover also lists 749 samples, while the methodology describes three samples per step per working-set tier. The report does not explain how those counts relate. Treat the timing and sample count as uncertain unless the raw run artifacts clarify them.

## Check your understanding

Why can a cold first fio sample be slower than later samples even if node pressure did not change? What does p99 add that a mean does not?

## Sources

- [fio documentation](https://fio.readthedocs.io/en/latest/fio_doc.html)
- [Linux memory-management concepts](https://docs.kernel.org/admin-guide/mm/concepts.html)
