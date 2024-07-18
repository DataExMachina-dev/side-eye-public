---
title: "Installing the Side-Eye agent"
weight: 10
---

## Installing the Side-Eye agent

In order to install `side-eye-agent` on a Linux machine:

- Log in to the [Side-Eye] web application and click on your user icon at the
  top right (or click [here](https://app.side-eye.dev/login)).
- Copy the API token corresponding to your organization.
  - token is shared by all users with email addresses at the same domain (see
    [Users and organizations]({{<relref "users-and-orgs">}}) for more info).
- Run `curl https://sh.side-eye.io | sh` and paste the API token when prompted.
  - Or pass the API token to the installation script directly:  
    `curl https://sh.side-eye.io | SIDE_EYE_API_TOKEN=<token> sh`

This will install the `side-eye-agent` through a snap package. Once agents are
running, they should show up when you log in to the Side-Eye web app at
[app.side-eye.io](https://app.side-eye.io).

[Side-Eye]: https://app.side-eye.io/

## Configuring Side-Eye to monitor processes

In order to take snapshots of your services, you first need to tell Side-Eye
which processes to monitor among all the processes running on the agents'
machines. These processes will then show up in the web application under the
_Capture snapshot_ button, available for selection.

Side-Eye monitors the processes that are assigned a "program". Programs are
logical names for what a process represents. For example, a process might
correspond to the CockroachDB program, or the Prometheus program. Side-Eye has a
flexible way of assigning processes to programs: program rules consist of
conjunctions of predicates. A predicate looks at some aspect of a process for
deciding if the condition passes; for example, it can look at the name of the
executable being used by the process. For example, we might configure it such
that every process using an executable named `cockroach` corresponds to the
CockroachDB program:

![image](program-rule-cockroach.png)

This configuration happens in the [Agents page](https://app.side-eye.io/agents) of
the web app.

Side-Eye uses the program configured for a process in a coupled of ways:

- The processes listed in the _Capture snapshot_ button are grouped by program
- The names of snapshots include all the programs of the processes that are part
  of the snapshot
- When visualizing goroutines in a snapshot, we can filter the goroutines by the
  program that the (process corresponding to the) goroutine belongs to
- Programs can be assigned _process friendly names_ in the spec: on the
  [Programs tab](https://app.side-eye.io/spec?tab=programs) of the spec we can
  specify how a process corresponding to a specific program should be identified
  in a snapshots (replacing the default identification by hostname/PID). For
  example, we can configure the "CockroachDB" program such that processes in a
  CockroachDB cluster are identified as "Node 1", "Node 2", etc. based on data
  dynamically read from each process' memory.
  See [Process friendly names]( {{<relref "process-friendly-names">}}) for more info.

## Configuring multiple distinct environments

An agent can be configured to label the processes it monitors as belonging to a
named "environment". This is useful when you want to segregate the
machines/processes into groups that should be monitored separately. For example,
you might have a `prod` and a `staging` environment, separating the respective
processes. Or, if you are running a multi-tenant service with per-tenant VMs,
you might want to group each tenant's machines/processes into separate
environments.

To configure the agent to use an environment, pass the `SIDE_EYE_ENVIRONMENT`
variable to the installation command. This command also works for updating the
parameters if the agent is already installed.
```shell
curl https://sh.side-eye.io | SIDE_EYE_API_TOKEN=<token> SIDE_EYE_ENVIRONMENT=<env name> sh`
```


## Programs versus binaries

TODO

## Programs versus modules

TODO
