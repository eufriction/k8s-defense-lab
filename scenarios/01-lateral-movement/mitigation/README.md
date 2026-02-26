# Mitigation: Block cross-namespace pod-to-pod traffic

This mitigation applies a backend ingress `CiliumNetworkPolicy` that denies all traffic to `backend/api`.

---

## Apply the ingress policy

```sh
kubectl apply -f scenarios/01-lateral-movement/mitigation/backend-ingress-policy.yaml
```

---

## Verify dropped cross-namespace traffic

```sh
kubectl exec -it web -n frontend -- zsh
curl -m 5 -sS http://api.backend.svc.cluster.local/get
```

The request should fail (timeout) instead of returning a response.

---

## Observe in Hubble (run on your machine in a second terminal)

```sh
cilium hubble port-forward &
hubble observe --from-pod frontend/web --verdict DROPPED --follow
```

While you run the `curl` command, Hubble should show dropped flows from `frontend/web` to `backend/api`.

---

## Apply the egress policy

The first request is still `FORWARDED`. This is the result of the applied policy only limiting ingress traffic to `backend`. Apply the following to also deny egress traffic from the `web` pod.

```sh
kubectl apply -f scenarios/01-lateral-movement/mitigation/frontend-egress-policy.yaml
```

## Verify dropped cross-namespace traffic

```sh
kubectl exec -it web -n frontend -- zsh
curl -m 5 -sS http://api.backend.svc.cluster.local/get
```

The result is that all traffic from the web pod is `DROPPED`. Not even DNS resolution works now.

## Roll back (optional for repeating the attack demo)

```sh
kubectl delete -f scenarios/01-lateral-movement/mitigation/backend-ingress-policy.yaml
```
