---
title: Side-Eye
type: docs
weight: 1
---

# Welcome to Side-Eye

Side-Eye is a next-generation debugger for production Go services.

A demo environment showing the Side-Eye monitoring a
[CockroachDB](https://www.cockroachlabs.com/) cluster is available at
[demo.app.side-eye.io](https://demo.app.side-eye.io).

## Getting started

There are two ways of monitoring a process with Side-Eye:
1. Install the Side-Eye agent on the process' machine (Linux on x86)
2. Include the [Side-Eye client library] into your Go program (Linux on x86,
   MacOS on arm64)

### Installing the agent

In order to install `side-eye-agent` on a Linux machine:

- Log in to the [Side-Eye] web application and click on your user icon at the
  top right (or click [here](https://app.side-eye.dev/login)).
- Copy the API token corresponding to your organization.
  - The token is shared by all users with email addresses at the same domain (see
    [Users and organizations]({{<relref "users-and-orgs">}}) for more info).
- Run `curl https://sh.side-eye.io | sh` and paste the API token when prompted.
  - Or pass the API token to the installation script directly:  
    `curl https://sh.side-eye.io | SIDE_EYE_API_TOKEN=<token> sh`

This will install the `side-eye-agent` through a snap package. Once agents are
running, they should show up when you log in to the Side-Eye web app at
[app.side-eye.io](https://app.side-eye.io).

See [Installing the Side-Eye agent]({{<relref "installing">}}) for more info.

### Using the Side-Eye client library

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

See [Side-Eye client library]({{<relref "client-library">}}) for more info.


If you're running a Kubernetes cluster, you can also deploy the `side-eye-agent` using a
helm chart. See [Deploying the Side-Eye agent in Kubernetes]({{<ref "kubernetes">}})
for more info.

[Side-Eye]: https://app.side-eye.io/
[Side-Eye client library]: https://github.com/DataExMachina-dev/side-eye-go
