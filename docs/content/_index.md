---
title: Ex-Ray
type: docs
weight: 1
---

# Welcome to Ex-Ray

Ex-Ray is a new-generation debugger for production Go services.

## Getting started

- Log in to the [Ex-Ray] web application and click on your user icon at the top right (or click [here](https://app.exray.dev/login)).
- Copy the API token corresponding to your organization.
  - This token is shared by all users with email addresses at the same domain (see [Users and organizations]({{<ref "users-and-orgs">}}) for more info).
- Run `curl https://sh.exray.dev | sh` and paste the API token when prompted.
  - Or pass the API token to the installation script directly:  
  `curl https://sh.exray.dev | sh - -s - -t <tenant-token>`

This will install the `ex-ray` agent through a snap package.

See [Installing the Ex-Ray agent]({{<ref "installing">}}) for more info.

[Ex-Ray]: https://app.exray.dev/
