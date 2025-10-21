Status:
```bash
kubectl -n kube-system get pods -l k8s-app=cilium
kubectl -n kube-system get deploy cilium-operator
kubectl -n kube-system exec ds/cilium -- cilium-dbg status --verbose
cilium connectivity test
```

Logs:
```bash
# Agent pods
kubectl -n kube-system get pods -l k8s-app=cilium -o name
kubectl -n kube-system logs --timestamps ds/cilium
kubectl -n kube-system logs --timestamps -p ds/cilium

# Operator
kubectl -n kube-system logs deploy/cilium-operator
kubectl -n kube-system logs -p deploy/cilium-operator
```

Configuration:
```bash
kubectl -n kube-system get cm cilium-config -o yaml | less
kubectl -n kube-system exec ds/cilium -- cilium-dbg config --all
kubectl -n kube-system exec pods/cilium-b45j9 -- cilium-dbg config --all
```

Services:
```bash
kubectl get svc -A
kubectl -n kube-system exec ds/cilium -- cilium-dbg service list
kubectl -n kube-system exec ds/cilium -- cilium-dbg bpf lb list
```

Monitor:
```bash
# Exec into that Cilium pod and monitor
kubectl -n kube-system exec -it <cilium-pod> -- cilium-dbg monitor --type drop
kubectl -n kube-system exec -it <cilium-pod> -- cilium-dbg monitor --type policy-verdict
kubectl -n kube-system exec -it <cilium-pod> -- cilium-dbg monitor -v

# Default sysdump
cilium sysdump
```
