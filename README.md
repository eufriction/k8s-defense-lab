# Introduction

This repository demonstrates how **Kyverno** and **Cilium Network Policies** protect a Kubernetes cluster when an attacker compromises an application. It provides a set of realistic attack scenarios, each paired with a mitigation, so you can observe the difference in behaviour before and after applying the controls.

---

## Prerequisites

- [mise](https://mise.jdx.dev/installing-mise.html)
- [Docker](https://docs.docker.com/get-docker/) (running locally)

`mise` manages all other required tools (`kind`, `helm`, `cilium-cli`, `hubble`, `kyverno`) automatically.

---

## Quick start

### Preparation (one-time)

#### Activate mise in your shell

Make sure `mise` is activated in your shell so the `mise run ...` tasks can load the right tools. Don’t skip this. Follow the official `mise activate` instructions for your shell:

- [mise activate documentation](https://mise.jdx.dev/cli/activate.html)

After adding the activation line, restart your shell or open a new terminal.

#### Set a GitHub token to avoid rate limits (recommended)

When `mise` installs tools hosted on GitHub, unauthenticated requests can hit GitHub API rate limits. Create a personal access token (no scopes needed for public tools) and set it for this project with `mise set --prompt`. See the GitHub token creation guide or go directly to the token creation page:

- https://docs.github.com/articles/creating-an-oauth-token-for-command-line-use
- https://github.com/settings/tokens/new

Set the token interactively:

```sh
mise set --prompt GITHUB_TOKEN
```

Avoid setting the token using `export GITHUB_TOKEN=token` or `mise set GITHUB_TOKEN` as that stores the token the history of your shell.

### Run the demo

#### Start the cluster

```sh
mise run cluster:start
```

This creates a `kind` cluster, installs Cilium in kube-proxy-free mode with Hubble observability, and installs Kyverno with Pod Security restrictions enabled.

#### Verify the cluster is healthy

```sh
mise run cluster:verify
```

#### Start the demo environment

```sh
mise run apps:start
```

This deploys a two-tier application across two namespaces:

| Pod   | Namespace  | Role                          |
| ----- | ---------- | ----------------------------- |
| `web` | `frontend` | Simulates a compromised app   |
| `api` | `backend`  | Simulates a sensitive backend |

#### Exec into the compromised pod

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

## Clean up

```sh
mise run apps:stop      # remove the demo workloads
mise run cluster:stop   # delete the cluster
```
