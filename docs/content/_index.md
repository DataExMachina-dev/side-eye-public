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

[Side-Eye]: https://app.side-eye.io/
