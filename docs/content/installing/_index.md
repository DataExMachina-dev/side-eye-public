---
weight: 1
bookFlatSection: true
title: "Installing the Ex-Ray agent"
---

## Installing the Ex-Ray agent

In order to install `exray-agent` on your machine:

- Log in to the [Ex-Ray] web application and click on your user icon at the top right (or click [here](https://app.exray.dev/login)).
- Copy the API token corresponding to your organization.
    - This token is shared by all users with email addresses at the same domain.
- Run `curl https://sh.exray.dev | sh` and paste the API token when prompted.
    - Or pass the API token to the installation script directly:  
      `curl https://sh.exray.dev | sh - -s - -t <tenant-token>`

This will install the `ex-ray` agent through a snap package.

[Ex-Ray]: https://app.exray.dev/
