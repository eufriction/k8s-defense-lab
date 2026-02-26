# k8s-defense-lab

This repository demonstrates how **Kyverno** and **Cilium Network Policies** protect a Kubernetes cluster when an application is compromised. It provides a set of realistic attack scenarios, each paired with a mitigation, so you can observe the difference in behaviour before and after controls are applied.

---

## Prerequisites

- [mise](https://mise.jdx.dev/installing-mise.html)
- [Docker](https://docs.docker.com/get-docker/) (running locally)

`mise` manages all other required tools (`kind`, `helm`, `cilium-cli`, `hubble`, `kyverno`) automatically.

---

## Quick Start

**1. Start the cluster**

```sh
mise run cluster:start
```

This creates a `kind` cluster, installs Cilium in kube-proxy-free mode with Hubble observability, and installs Kyverno with Pod Security restrictions enabled.

**2. Verify the cluster is healthy**

```sh
cilium status --wait
kubectl get pods -n kyverno
```

**3. Start the demo environment**

```sh
mise run demo:start
```

This deploys a two-tier application across two namespaces:

| Pod   | Namespace  | Role                          |
| ----- | ---------- | ----------------------------- |
| `web` | `frontend` | Simulates a compromised app   |
| `api` | `backend`  | Simulates a sensitive backend |

**4. Exec into the compromised pod**

```sh
kubectl exec -it web -n frontend -- zsh
```

You are now inside the attacker's foothold. Head to `scenarios/` to run through the attacks.

---

## Scenarios

Each scenario has an `attack/` and a `mitigation/` directory with a `README.md` that walks you through what to run and what to observe.

---

## Observing with Hubble

Open the Hubble UI to watch network flows in real time as you run each scenario:

```sh
cilium hubble ui
```

Or observe from the command line:

```sh
cilium hubble port-forward &
hubble observe --namespace frontend --follow
hubble observe --namespace backend --follow
```

---

## Teardown

```sh
mise run demo:stop      # remove the demo workloads
mise run cluster:stop   # delete the cluster
```
