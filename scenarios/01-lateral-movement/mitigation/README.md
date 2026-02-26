# Mitigation: Block cross-namespace pod-to-pod traffic

This mitigation applies a backend ingress `CiliumNetworkPolicy` that denies all traffic to `backend/api`.

---

## Apply the policy

```sh
kubectl apply -f scenarios/01-lateral-movement/mitigation/backend-ingress-policy.yaml
```

---

## Verify dropped cross-namespace traffic

```sh
kubectl exec web -n frontend -- curl -m 5 -sS http://api.backend.svc.cluster.local/get
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

## Roll back (optional for repeating the attack demo)

```sh
kubectl delete -f scenarios/01-lateral-movement/mitigation/backend-ingress-policy.yaml
```
