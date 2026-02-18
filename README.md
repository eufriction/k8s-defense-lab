# k8s-defense-lab

This repository aims to show how Kyverno Policies and Cilium Network Policies is used to protect a Kubernetes Cluster from being compromised if an application gets compromised. The repository contains the following:

- a configuration file for `kind` that is compatible with Cilium CNI and Service Mesh
- Cilium with Cilium Network Policies in Audit-mode
- Kyverno running with Pod Secuirty Restrictions and Policy Exceptions enabled

The repository use `mise` to manage the development environment. Although it's possible to install tools in many ways, this is the only supported way. Check the [mise documentation](https://mise.jdx.dev/installing-mise.html) for instructions on how to install `mise`.
