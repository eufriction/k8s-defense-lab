# Attack: Cross-namespace lateral movement

## What this does

1. Start from the compromised `frontend/web` pod
1. Reach the `backend/api` pod through its service
1. Show dropped cross-namespace pod-to-pod traffic

---

## Prepare the demo pods (run on your machine)

```sh
mise run apps:start
kubectl delete -f scenarios/01-lateral-movement/mitigation/backend-ingress-policy.yaml --ignore-not-found
kubectl delete -f scenarios/01-lateral-movement/mitigation/frontend-egress-policy.yaml --ignore-not-found
kubectl exec -it web -n frontend -- zsh
```

All remaining commands run inside this shell.

---

## Observe in Hubble (run on your machine in a second terminal)

```sh
cilium hubble port-forward &
hubble observe --from-pod frontend/web --follow
```

While you run the `curl` command, Hubble should show `FORWARDED` flows from `frontend/web` to `backend/api`.

---

## Reach the backend pod from the frontend pod

```sh
curl -sS http://api.backend.svc.cluster.local/get
```

Both requests should return responses, showing unrestricted cross-namespace traffic.
