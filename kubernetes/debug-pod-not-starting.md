# Debugging a pod that won't start

A checklist I go through, in this order.

## 1. What state is it in?

```bash
kubectl get pods -n <ns> -o wide
```

| Status | Usually means |
|---|---|
| `Pending` | Scheduler can't place it: no resources, node selector / taints, unbound PVC |
| `ImagePullBackOff` / `ErrImagePull` | Wrong image name/tag or missing registry credentials |
| `CrashLoopBackOff` | Container starts and exits: app error, bad config, failing liveness probe |
| `CreateContainerConfigError` | Missing ConfigMap / Secret referenced in the spec |
| `OOMKilled` (in last state) | Memory limit too low |

## 2. Events tell you why

```bash
kubectl describe pod <pod> -n <ns>      # read the Events section at the bottom
kubectl get events -n <ns> --sort-by=.lastTimestamp | tail -20
```

## 3. Logs, including the previous crash

```bash
kubectl logs <pod> -n <ns>
kubectl logs <pod> -n <ns> --previous   # logs of the container that just crashed
kubectl logs <pod> -n <ns> -c <container>   # multi-container pods
```

## 4. Get inside

```bash
kubectl exec -it <pod> -n <ns> -- sh
# distroless / no shell? attach a debug container:
kubectl debug -it <pod> -n <ns> --image=busybox --target=<container>
```

## 5. Check what the pod actually received

```bash
kubectl get pod <pod> -n <ns> -o yaml | less    # final spec after admission/mutation
kubectl exec <pod> -n <ns> -- env | sort         # environment variables
```

## Common fixes

- **Readiness probe fails but app is fine** → wrong path/port, or `initialDelaySeconds` too short.
- **CrashLoop right after deploy** → compare config with the last working ReplicaSet:
  `kubectl rollout history deploy/<name>` and `kubectl rollout undo deploy/<name>` to roll back fast.
- **Pending with "Insufficient cpu"** → requests are too high for the nodes; lower requests or add capacity.
