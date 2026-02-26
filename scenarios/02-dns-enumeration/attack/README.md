# Attack: DNS enumeration

## Scenario

After gaining shell access to the `web` pod, an attacker uses the cluster's internal DNS resolver to map the topology of the cluster—discovering services, namespaces, and potential targets before moving laterally.

## Prerequisites

The demo environment must be running:

```sh
mise run apps:start
```

Exec into the compromised pod:

```sh
kubectl exec -it web -n frontend -- zsh
```

Run all commands below from inside this shell.

---

## Steps

### Confirm the local service resolves

```sh
dig web.frontend.svc.cluster.local
```

### Discover the backend service

```sh
dig api.backend.svc.cluster.local
```

### Walk well-known namespaces looking for exposed services

```sh
for ns in default kube-system kyverno monitoring ingress-nginx cert-manager; do
  echo "=== $ns ==="
  dig kubernetes.$ns.svc.cluster.local +short
done
```

### Reverse-resolve Kubernetes Service IPs to infer naming conventions

```sh
BACKEND_SVC_IP=$(dig +short api.backend.svc.cluster.local)
dig -x $BACKEND_SVC_IP
```

### Attempt a wildcard lookup to enumerate services in a namespace

```sh
# DNS does not support wildcards natively, but common service names can be brute-forced
for svc in api web app backend frontend db redis postgres mysql kafka rabbitmq; do
  result=$(dig +short $svc.backend.svc.cluster.local)
  [ -n "$result" ] && echo "$svc.backend → $result"
done
```

---

## What this demonstrates

Without DNS egress restrictions, a compromised pod can freely query the cluster DNS server to:

- Enumerate every service across every namespace it can guess or iterate
- Build a map of the internal network topology before launching targeted attacks
- Identify high-value targets such as databases, message queues, or privileged system services
