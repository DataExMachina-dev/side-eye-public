---
title: Side-Eye
type: docs
weight: 1
---

# Welcome to Side-Eye

Side-Eye is a new-generation debugger for production Go services.

## Getting started

In order to install `side-eye-agent` on your machine:

- Log in to the [Side-Eye] web application and click on your user icon at the top right (or click [here](https://app.side-eye.dev/login)).
- Copy the API token corresponding to your organization.
  - token is shared by all users with email addresses at the same domain (see
    [Users and organizations]({{<ref "users-and-orgs">}}) for more info).
- Run `curl https://sh.side-eye.io | sh` and paste the API token when prompted.
  - Or pass the API token to the installation script directly:  
    `curl https://sh.side-eye.io | SIDE_EYE_API_TOKEN=<token> sh`

This will install the `side-eye-agent` through a snap package.

See [Installing the Side-Eye agent]({{<ref "installing">}}) for more info.

[Side-Eye]: https://app.side-eye.io/
