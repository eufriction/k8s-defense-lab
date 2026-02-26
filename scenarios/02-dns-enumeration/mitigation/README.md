# Mitigation: DNS enumeration

## Strategy

Restrict outbound DNS queries from the `web` pod using a Cilium L7 DNS policy.
Instead of allowing the pod to query any name, the policy creates an allowlist of
DNS names the pod is legitimately permitted to resolve. Any query that does not
match the allowlist gets dropped by the Cilium DNS proxy before it ever reaches
CoreDNS.

---

## Apply the policy

```sh
kubectl apply -f scenarios/02-dns-enumeration/mitigation/cilium-network-policy.yaml
```

---

## What the policy does

```yaml
# cilium-network-policy.yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: web-restrict-dns
  namespace: frontend
spec:
  endpointSelector:
    matchLabels:
      app: web
  egress:
    - toEndpoints:
        - matchLabels:
            io.kubernetes.pod.namespace: kube-system
            k8s-app: kube-dns
      toPorts:
        - ports:
            - port: "53"
              protocol: UDP
            - port: "53"
              protocol: TCP
          rules:
            dns:
              - matchName: "api.backend.svc.cluster.local"
```

The policy allows the `web` pod to:

- Send DNS queries **only** to CoreDNS in `kube-system`
- Resolve **only** `api.backend.svc.cluster.local`

The Cilium DNS proxy silently drops all other DNS queries—including attempts to
enumerate `kube-system`, `kyverno`, or any other namespace.

---

## Observe with Hubble

```sh
cilium hubble port-forward &
cilium hubble ui &
hubble observe --namespace frontend --verdict dropped --follow
```

Dropped DNS flows appear as:

```
frontend/web → kube-system/coredns  UDP 53  dropped
```

---

## Verify

Exec into the `web` pod and confirm the allowlisted name still resolves:

```sh
kubectl exec -it web -n frontend -- zsh
dig api.backend.svc.cluster.local
```

Confirm that Cilium blocks all other names:

```sh
kubectl exec -it web -n frontend -- zsh
dig kubernetes.default.svc.cluster.local
dig kubernetes.kube-system.svc.cluster.local
```

Both blocked queries should return `NXDOMAIN` or time out with no answer.

---

## Switching from policy evaluation mode

This demo runs Cilium in enforcing mode. To make the policy audit rather than
dropping, update the Helm values and re-install:

```yaml
# cilium/values.yaml
policyAuditMode: true
```

```sh
helm upgrade cilium cilium/cilium \
  --namespace kube-system \
  --version $CILIUM_VERSION \
  --values cilium/values.yaml \
  --set k8sServiceHost=$APISERVER_IP \
  --set k8sServicePort=6443
```

---

## Roll back

```sh
kubectl delete -f scenarios/02-dns-enumeration/mitigation/cilium-network-policy.yaml
```
