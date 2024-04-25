---
title: "Snapshots"
weight: 1
---

# Snapshots

## What are snapshots?

Side-Eye's main feature is the ability to "capture snapshots" of your distributed
system. A snapshot contains information about the distributed state of execution
of a service at a point in time; it includes the stack traces of all the
goroutines currently alive across all the processes that are part of the
snapshot (both goroutines that are currently running on-CPU, and the many more
goroutines that are blocked waiting for some asynchronous condition), plus
variable data that was captured from different stack frames (i.e. functions
currently being executed by some goroutine).

Snapshots are similar in spirit to core dumps. However, they are much more
targeted and can be collected way cheaper. A snapshot does not include the whole
heap; it includes only data that was explicitly requested. This makes snapshots
cheap enough to be collected at will. Snapshots bring the captured stack traces
front and center, so they are also similar to the "goroutine profiles" that
pprof can produce for Go processes (not to be confused with the "cpu profiles"
produced by pprof; see [comparing to profiling]({{< ref
"faq#how-does-side-eye-compare-to-profiling" >}}) for more), except that snapshots
also include variable data. And, of course, both core dumps and goroutine
profiles are associated with a single process whereas snapshots work across many
processes.

When visualizing a snapshot with the [Side-Eye web app](https://app.side-eye.io), you can
focus on the stack traces, or on specific variable data, or you can visualize
the two inter-mixed. Side-Eye aims to be the best goroutine exploration tool out
there for low-level spelunking, but it also lets you create and focus on
higher-level "reports" out of the variable data.

## Exploring the goroutines/stack traces in a profile

TODO

## Exploring the variable data in a profile

TODO

## Safety guarantees

Taking a snapshot is "safe": it is guaranteed to not affect the execution of
target processes (apart from very briefly pausing them; see below). Side-Eye uses
eBPF probes, which are verified by the Linux kernel to not have side effects
(e.g. they cannot modify the state of the target process in any way; they can't
cause segmentation faults, etc.).

## Technical details

A snapshot can include data from multiple processes. This data is collected as
close together in time as possible, but there is no explicit synchronization
between the different processes (i.e. different processes are resumed as soon as
their individual data has been collected, without blocking for the others). As
such, small amounts of skew is possible between the collection times of
different processes. This means that you can observe, for example, a server-side
RPC handler running on process A without a corresponding client on process B
(because process' B data was collected later than A's).

## What is the cost of a snapshot?

A process is paused while its data is collected. How long that pause is depends
on how many goroutines a process has and how much data Side-Eye was asked to
collect. Generally it takes from under 1ms to 3ms. If this is too much for you,
please contact us.
