---
title: "Deploying the Side-Eye agent in Kubernetes"
weight: 1
---

# Deploying the Side-Eye agent in Kubernetes

## Prerequisites

- A Kubernetes cluster
- `kubectl` CLI
- `helm` CLI hooked up to that cluster

## Install the Side-Eye agent as a DaemonSet via Helm

#### Create a secret with your Side-Eye API token

```sh
kubectl create secret generic side-eye-agent-token --from-literal=token=<your-api-token>
```

#### Install the Helm chart

```sh
helm install side-eye \
  oci://us-docker.pkg.dev/data-ex-machina/side-eye-agent-helm/side-eye-agent \
  --set environment=<your-environment-name>
```
