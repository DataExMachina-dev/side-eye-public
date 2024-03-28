---
title: "Installing the Ex-Ray agent"
weight: 1
---

## Installing the Ex-Ray agent

In order to install `exray-agent` on your machine:

- Log in to the [Ex-Ray] web application and click on your user icon at the top right (or click [here](https://app.exray.dev/login)).
- Copy the API token corresponding to your organization.
    - token is shared by all users with email addresses at the same domain (see
      [Users and organizations]({{<ref "users-and-orgs">}}) for more info).
- Run `curl https://sh.exray.dev | sh` and paste the API token when prompted.
    - Or pass the API token to the installation script directly:  
      `curl https://sh.exray.dev | sh - -s - -t <tenant-token>`

This will install the `ex-ray` agent through a snap package.

[Ex-Ray]: https://app.exray.dev/

## Configuring Ex-Ray to monitor processes

In order to take snapshots of your services, you first need to tell Ex-Ray which
processes to monitor among all the processes running on the agents' machines.
These processes will then show up in the web application under the _Capture
snapshot_ button, available for selection.

Ex-Ray monitors the processes that are assigned a "program". Programs are
logical names for what a process represents. For example, a process might
correspond to the CockroachDB program, or the Prometheus program. Ex-Ray has a
flexible way of assigning processes to programs: program rules consist of
conjunctions of predicates. A predicate looks at some aspect of a process for
deciding if the condition passes; for example, it can look at the name of the
executable being used by the process. For example, we might configure it such
that every process using an executable named `cockroach` corresponds to the
CockroachDB program:

![image](program-rule-cockroach.png)

This configuration happens in the [Agents page](https://app.exray.dev/agents) of
the web app.

Ex-Ray uses the program configured for a process in a coupled of ways:
- The processes listed in the _Capture snapshot_ button are grouped by program
- The names of snapshots include all the programs of the processes that are part
  of the snapshot
- When visualizing goroutines in a snapshot, we can filter the goroutines by the program that the (process corresponding to the) goroutine belongs to
- Programs can be assigned _process friendly names_ in the spec: on the
  [Programs tab](https://app.exray.dev/spec?tab=programs) of the spec we can
  specify how a process corresponding to a specific program should be identified
  in a snapshots (replacing the default identification by hostname/PID). For
  example, we can configure the "CockroachDB" program such that processes in a
  CockroachDB cluster are identified as "Node 1", "Node 2", etc. based on data
  dynamically read from each process' memory.
  See [Process friendly names]( {{<ref "process-friendly-names">}}) for more info.
  

## Configuring multiple distinct environments

TODO

## Programs versus binaries

TODO

## Programs versus modules

TODO
