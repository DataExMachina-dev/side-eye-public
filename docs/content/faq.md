---
title: "FAQ"
weight: 1
---

# FAQ

### Is Ex-Ray free to use?

Yes.

### How does Ex-Ray differ from a traditional debugger like Delve or GDB?

Ex-Ray shares some of its basic capabilities with traditional debuggers, but
differs from them both in its mode of usage, and in the underlying technology.
Philosophically, Ex-Ray is focused on server software running in production
environments, services with larger footprints (multiple microservices executing
different binaries on multiple machines), and teams of people debugging a
service over time. A traditional debugger has a narrower focus stemming from the
desktop software era — a single person debugging a single process that they can
pause and resume at will, with every session starting from scratch every time.

A debugger like Delve provides features like breakpoints and reading variables
out of a target process' memory. You generally use Delve either through its
command line interface, or through your editor. Every Delve session starts with
a "blinking cursor" — the debugger says nothing until you ask it very specific
questions. The usual interaction is that you stop the process, think for a
while, look at some data, resume it, and maybe stop it again later to look at
more data.

Ex-Ray has some of the same technical abilities to interact with processes that
are not particularly cooperating with the debugger, but they are packaged
differently:
1. You're generally not debugging a single process (although you certainly can
   use it on a single process); you're debugging many different processes at
   once, running on different machines.
2. You benefit from your colleagues having previously used Ex-Ray to instrument
   the programs you're looking at: all the data that they've asked for is
   automatically collected for you to peruse.
3. Ex-Ray is all about raising the level of abstraction that you're supposed to
   operate on when debugging your service. Data is collected automatically for
   you across goroutines and functions so that you don't have to ask for it
   variable by variable. Then, data can be further organized into SQL tables
   which can act as "reports". For example, when debugging a server, you might
   build a table of all the requests or queries that are currently executing.
   Many debugging sessions might start by looking at this table instead of
   wandering aimlessly through goroutines. The oldest running request might be
   interesting, for example, at which point you might drop from that request's
   details onto lower level things — for example by navigating to that query's
   goroutine to look at its exact state of execution.
4. Being focused on dynamic, heavily concurrent and parallel software,
   Ex-Ray has particular features for exploring goroutine stack traces: every
   Ex-Ray snapshot includes the stack traces of all the goroutines and they can
   be inspected graphically.
5. Ex-Ray does not "pause" the target processes for any measurable amount of
   time (capturing a snapshot takes on the order of a millisecond), and there's
   certainly no human in the loop while a target process is paused. You interact
   with your service by taking a snapshot, letting the service continue
   unperturbed, and then later maybe taking another snapshot asking for more
   data.

At a technical level, Ex-Ray uses the same debug information produced by the
compiler that Delve does in order to find the locations of variables and make
sense of a process' memory. However, the actual interaction with a target
process works differently. Most debuggers use the ptrace() system call to
interact with the target process in a piecemeal way: one ptrace() call to read a
pointer value from the target memory space, then another call to dereference
that pointer, etc. Ex-Ray works differently, for much greater efficiency: based
on the data that Ex-Ray was asked to collect, it uses the debug info to generate
a eBPF probe that it injects into the target process. See more details [below](#how-does-ex-ray-work).

### What are snapshots?

Snapshots are currently Ex-Ray's main feature. A snapshot contains information
about the distributed state of execution of a service at a point in time; it
includes the stack traces of all the goroutines currently alive across all the
processes that are part of the snapshot (both goroutines that are currently
running on-CPU, and the many more goroutines that are blocked waiting for some
asynchronous condition), plus variable data that was captured from different
stack frames (i.e. functions currently being executed by some goroutine).

Snapshots are similar in spirit to core dumps. However, they are much more
targeted and can be collected way cheaper. A snapshot does not include the whole
heap; it includes only data that was explicitly requested. This makes snapshots
cheap enough to be collected at will. Snapshots bring the captured stack traces
front and center, so they are also similar to the "goroutine profiles" that
pprof can produce for Go processes (not to be confused with the "cpu profiles"
produced by pprof; see [below](#how-does-ex-ray-compare-to-profiling)), except
that snapshots also include variable data. And, of course, both core dumps and
goroutine profiles are associated with a single process whereas snapshots work
across many processes.

When visualizing a snapshot with the [Ex-Ray web app](app.exray.dev), you can
focus on the stack traces, or on specific variable data, or you can visualize
the two inter-mixed. Ex-Ray aims to be the best goroutine exploration tool out
there for low-level spelunking, but it also lets you create and focus on
higher-level "reports" out of the variable data.

### How does Ex-Ray work?

Ex-Ray is made out of a control-plane that we (Data Ex Machina Debugging
Company) run as a hosted service, and an agent that you need to run on every
machine where any processes to be monitored are executing. Ex-Ray is used
through a [web app](app.exray.dev). When you ask the web app to snapshot a bunch
of processes, the Ex-Ray control plane looks at your specification of what data
to collect, asks the agents to retrieve the debug information of all the
binaries are used by the processes to snapshot, and dynamically generates two
programs for each one of the binaries: a eBPF probe and a user-space process
that processes the data produced by the probe.

The generated eBPF probes know how do to a couple of things:
- iterate through all the live goroutines and walk their stacks
- while walking the stacks, recognize functions for which any data is to be collected
- for such a function, locate the required variables in memory depending on the
  program counter corresponding to that function
- read the respective variables from memory and copy them to an output buffer
- besides collecting full variables, Ex-Ray can also "evaluate" expressions like
  `foo.bar.baz` consisting of accessing struct fields and dereferencing pointers
- the probe understand a various go data structured; for example, it can
  dereference interfaces and figure out what type hides behind the interface box

The generated user-space processes decode the raw data produced by the probes
and turn it into JSON. They go from the raw bytes produced for a stack frame to
variables which have fields, recursively.

The control plane distributed the generated programs to the respective agents,
which attach the eBPF probe to the target process, and load the user-space
program as a Web Assembly module (for isolation purposes). Each agent then
briefly pauses its target processes, triggers the probe, processes the output
data and returns it to the control plane. Back on the server, we put the data
from all agents together and save it as a snapshot.

### Is taking a snapshot safe?

Yes! Taking a snapshot is guaranteed to not affect the execution of target
processes (apart from very briefly pausing them; see below). Ex-Ray uses eBPF
probes, which are verified by the Linux kernel to not have side effects (e.g.
they cannot modify the state of the target process in any way; they can't cause
segmentation faults, etc.).

### How do I explore and organize the snapshot data that Ex-Ray collects?

For now at least, each snapshot is consumed in isolation — there is no built-in
way to look at data across snapshots. Within a snapshot, data is organized in
SQL tables that the web app lets you interact with. Also, all the data in a
snapshot can be downloaded as a SQLite database file, and queried on your
machine however you see fit. There are two types of tables: _function tables_
and _derived tables_.

A _function table_ is created for every function in your spec — i.e. every
function for which any variables/expressions are to be captured. These tables
get a row for every stack frame (across all goroutines) that corresponds to the
table's function. Every variable/expression to be captured is a column in the
table. If the value of one of the expressions is a Go struct, then the
corresponding column will contain JSON documents. If a variable is not available
to be collected on a particular frame (because it is no longer alive at the
program counter where the snapshot has caught the respective frame, or because
the compiler has optimized it away or the debug information is faulty), the
respective column will be NULL.

Besides the columns corresponding to variables/expressions, function tables can
also have "extra columns" defined by SQL expressions evaluated on top of the
variable columns. A common thing to do is to use a JSON Path expression that
extracts a field from a variable into its own, dedicated column. For example, if
there's a variable called `foo` being captured of struct type (resulting in a
column also called `foo`), an extra column with the expression `foo->bar.baz`
will extract the respective field from `foo`'s JSON.

A _derived table_ is defined by a SQL query; the query has access to the
function tables and to all the derived tables previously defined. The query is
arbitrary — it can join, filter and aggregate data as it sees fit. For example,
you might imagine a derived tables that joins together data from multiple frames
of a single goroutine (e.g. putting together information about a network
connection known by a function towards the bottom of the stack) and information
about the current query being served on that connection (known by a function
down the stack). Or, you might imagine a table that joins data across goroutines
— for example producing a report about which operation is blocked on which other
operation.

The point of the derived tables is to raise the level of abstraction that you're
operating on when debugging your service: start from high-level information that
someone has already aggregated for you instead of looking at raw goroutine data.

An exciting Ex-Ray feature is around linking of table rows. You can define pairs
of tables (and also specifically pairs for frames) whose rows should be linked
based on common values in a set of columns. To use CockroachDB as a good
example, SQL queries execute within transactions, which are identified by a
transaction id. Transactions can hold database locks, which can block other SQL
queries. A query blocked on acquiring a lock naturally knows the transaction id
of the lock holder. In Ex-Ray, we can link (frames belonging to) the blocked
function with frames corresponding to executing the transaction with the same
id. If we do this, when looking at the goroutine corresponding to a blocked
query, we can easily navigate to the goroutine corresponding to the transaction
holding the lock and see what it's currently doing. This can be very effective
during an investigation, and it's something that's very hard to do without Ex-Ray.

To "raise the level of abstraction" of debugging sessions even more, we're
prototyping a Grafana plugin that allow the creation of dashboards powered by
Ex-Ray data. The idea is to click and drag your way to nice-looking web pages
that present and summarize our SQL tables, drawing attention to unusual things.
For example, one can image a dashboard listing the oldest active queries in the
system, coloring the ones that are older than some threshold in red.

### Where is the snapshot data stored?

All the data is stored by Data Ex Machina's hosted service, "in the cloud". We
recognize that this might not be palatable for some users; please contact us at
[contact@dataexmachina.dev](mailto:contact@dataexmachina.dev).

### How does Ex-Ray compare to profiling?

This is a trick question because Ex-Ray snapshots are partly a form of
profiling, by summarizing the state of execution of all the goroutines at a
point in time (pprof calls this a "goroutine profile"). Most people are thinking
about "CPU profiling" though — a software analysis technique whereby you sample
the stack of the goroutine currently executing on a CPU core (if any) at some
frequency (e.g. 100 Hz) over some time duration (or continuously) and then
produce a report that aggregates these samples.

CPU profiling is very useful for understanding what code is "hot" — which
functions are using the most resources. A popular way to visualize the
collection of samples in a CPU profile is through a flame graph (Ex-Ray also
uses flame graphs to visualize the stacks of the goroutines in a snapshot;
perhaps that's what prompted this question). Such a report can tell you where to
invest in code optimization because it's likely to turn out into savings on your
cloud compute bill, or on higher throughput. At times, a CPU profile can be very
useful for debugging too, as it can indirectly hint to some pathology that a
process has hit (e.g. perhaps a quadratic algorithm has finally bitten a user,
or perhaps an error code path that you weren't expecting to run at all is
suddenly very prominent).

On the other hand, CPU profiles generally don't have any "data" in them: they
only describe the shape of the code that is executing without telling you if
you're looking at a single request being executed over a long time duration or
many short requests, or which particular user of a multi-tenant system is
responsible for the CPU load. Also, by definition, CPU profiles only give you
information about what's running on a CPU; they don't tell you anything about
blocked operations. So, for example, if all your operations are mostly blocked),
a CPU profile may well be completely empty. Or, to the contrary, if you're
profiling a trading system, you might see that the CPU is always completely busy
doing uninteresting work — polling the network interface. Because of all these,
CPU profiles are not really a debugging tool.

Snapshots are not a statistical measurement and they do not rely on any
sampling: they contain information about all the goroutines alive at a point in
time (the flip side is that a snapshot only talks about a single point in time).
Similarly, snapshots do not make a distinction between goroutines that are
running when the snapshot was taken and goroutines that are not currently
scheduled (because they are blocked, or simply because all threads are busy) —
you get information on all of them just the same. And, of course, snapshots
include arbitrary data that you ask for, which means you can find out exactly
what your program is working on. This is difference, for example, between a CPU
profile telling you that your database is busy with executing SQL queries, and
telling you specifically which queries it is currently executing (including
telling you whether there's one huge query executing in parallel on a bunch of
goroutines or if you're dealing with many small queries).

Having said all this about how great snapshots are, trying to figure out
specifically "what is using my CPUs" is a common question, and one can imagine a
debugger benefiting from features in this area.

### Are other languages besides Go supported?

Not at the moment. A lot of the basic technology we've built is not specific to
Go, but the product only focuses on Go so far. If you'd like to see another
language or a specific environment supported, please let us know at
[contact@dataexmachina.dev](mailto:contact@dataexmachina.dev).

### What are the technical requirements for using Ex-Ray?

The Ex-Ray agent currently only works on Linux, because it uses eBPF and because
we only deal with debug information in the DWARF format. The agent needs to be
run with admin privileges because of eBPF.


### Does it only work on Linux?

Yes, for now. See [above](#what-are-the-technical-requirements-for-using-ex-ray).

### What features are coming to Ex-Ray next?

Ex-Ray wants to be a debugger for the cloud era, and there's certainly a lot of
features that one might want from a dream debugger beyond snapshots. So there's
a lot more we want to do.

Ex-Ray wants to be in the dynamic instrumentation game, and perhaps the most
obvious application of dynamic instrumentation is the dynamic injection of new
logs and metrics into an otherwise not instrumented process (i.e. "every time
execution reaches this line of code, output a log message and include this
expression and this other expression" or "every time execution reaches this line
of code, increment a counter and plot the rate of change of this counter"). So
we're definitely always thinking about these; they'll come soon.

Some common monitoring and debugging tasks revolve around understanding rare
errors or tail latency. Here, you would like to get your hands on some type of
trace of the operation that exhibited the bad behavior. It'd be cool if you
could say "next time this error happens on any of my machines, capture a stack
trace, include these variables and make it available somewhere". Or, next time
an operation was slow, give me these set of verbose logs from the relevant
goroutine. Tracing as a general concept is something we think about a lot. Go's
runtime traces have been improved recently and can help latency investigations,
so we're also considering how to surface them.

Anyway, if you have thoughts, please tell us at
[contact@dataexmachina.dev](mailto:contact@dataexmachina.dev).

### This all sounds great! How can I help?

Try Ex-Ray out and give us feedback at
[contact@dataexmachina.dev](mailto:contact@dataexmachina.dev). Or write to us
about your observability / monitoring / debugging needs. If you have any
thoughts on the topic, we'd definitely want to hear from you.
