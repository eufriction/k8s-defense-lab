# Mitigation: Container escape

This mitigation blocks the two fields used by the escape:

- `hostPID: true`
- `securityContext.privileged: true`

---

## Apply policy

```sh
kubectl apply -f kyverno-policy.yaml
```

---

## Verify validation fails

```sh
kubectl apply -k scenarios/06-container-escape/attack/
```

Kyverno should block the patched `web` pod with an admission denial:

```
Error from server: error when creating "...": admission webhook "validate.kyverno.svc-fail" denied the request:
policy Pod/frontend/web for resource violations:
  restrict-privileged-containers:
    deny-privileged-containers: Privileged containers and hostPID are not allowed.
```

---

## Roll back (optional for repeating the attack demo)

```sh
kubectl delete -f kyverno-policy.yaml
```
