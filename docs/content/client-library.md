---
title: "Side-Eye client library"
weight: 60
---

# The Side-Eye client library

An alternative to deploying [the Side-Eye agent]({{<relref "installing">}}) on a
host for monitoring processes running on that machine, programs can be monitored
with Side-Eye if they import the Side-Eye client library. This Go library can be
used for two purposes:
1. Allow programs to be monitored by Side-Eye (thus replacing the need for the
   agent). The agent currently only works on x86 Linux, whereas the library works
   on both x86 Linux and MacOS on arm64.
2. Perform API calls to Side-Eye, for example for triggering snapshots.

## Using the library as an agent replacement

To use the library as a replacement for the agent:

- Add a dependency to the library to your program:
  ```shell
  go get github.com/DataExMachina-dev/side-eye-go
  ```
- Configure the library to connect to the Side-Eye cloud service:
  ```go
  import "github.com/DataExMachina-dev/side-eye-go/sideeye"
  ...
  if err := sideeye.Init(context.Background(), "<name of my program/service>"); err != nil {
    log.Printf("failed to init Side-Eye: %s", err)
  }
  ```
  The `<name of my program/service>` argument should correspond to the program
  name under which processes running this source code should be grouped by
  Side-Eye.

  Once the connection to Side-Eye is established, the respective process appears
  both in the agents list on [app.side-eye.io](https://app.side-eye.io) and in the
  monitored processes lists.

  The Side-Eye client needs to be configured with an API token in order talk to
  the cloud service. This token can either be specified through the
  `SIDE_EYE_TOKEN` environment variable, or, alternatively, it can be passed
  configured in the source code by passing the `sideeye.WithToken("<my org's
  token>")` option to the `Init()` call. See the [Installing the
  agent](#installing-the-agent) section above for instruction on finding out the
  API token corresponding to your organization.

  Calling `Init()` to automatically establish a connection to the cloud service is
  not strictly necessary. The configuration page can be used at run-time to
  establish a connection manually, on-demand; see below.
- \[Optional\] Install the HTTP handler to expose a configuration page:
  ```go
	httpL, err := net.Listen("tcp", "localhost:8080")
	if err != nil {
		panic(err)
	}
	go func() {
		http.Handle("/sideeye", sideeye.HttpHandler())
		log.Fatal(http.Serve(httpL, nil))
	}()
  ```
  This snippet will install an HTTP handler accessible at
  `http://localhost:8080/sideeye` that generates a configuration page allowing
  users to connect or disconnect from the Side-Eye cloud service and configure the
  connection parameters.

### Environments

The `Init()` call can be passed the `WithEnvironment(<env name>)` option in
order to make the current process be part of the specified environment. If the
`WithEnvironment` option is not specified, the library looks for the
`SIDE_EYE_ENVIRONMENT` env var and, if it is set, uses its value as the
environment name. If the env var is not defined, the process will not be part of
any environment (i.e. the process will appear in the list of processes without
an environment in Side-Eye).

The use of environments with the client library should be done similarly to
their use in agents (see [configuring the agent]({{<relref
"installing#configuring-multiple-distinct-environments">}})). For example, you
might want to group processes into a `prod` and a `staging` environment.

### Library implementation details

The client library cooperates with the Side-Eye cloud service in order to
monitor the respective processes. A network connection is established to
side-eye.io (through an `Init()` call or through a user action on the
configuration page). If a snapshot or other instrumentation of a process is
required, the binary's debug information is uploaded to side-eye.io (if Side-Eye
has not seen that binary before) based on which Side-Eye produces instructions
("programs", if you will) describing how to find variables in memory and how to
read their data. Side-Eye sends these programs back to the library, which then
executes them. The exfiltrated data is sent back to Side-Eye, where it can be
visualized and consumed similarly to the data collected by agents.

## Using the library for making API calls to Side-Eye

The `sideeyeclient` package in the `github.com/DataExMachina-dev/side-eye-go/sideeye` module
can be used to trigger the snapshotting of all monitored processes in an environment.

## Limitations

The library does not currently support producing dynamic events.
