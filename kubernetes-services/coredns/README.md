# CoreDNS in Kubernetes

## 1. What is CoreDNS?
CoreDNS is a flexible, extensible DNS server that serves as the default internal DNS service for Kubernetes clusters (deployed as a Deployment in the `kube-system` namespace).

## 2. Why Kubernetes Uses CoreDNS
- **Dynamic Service Discovery**: As Pods are created, rescheduled, or scaled, their ephemeral IPs change constantly. CoreDNS ensures clients connect to Services via stable DNS names without hardcoding IP addresses.
- **Lightweight & Fast**: Implemented in Go with low memory consumption and high throughput.
- **Plugin Architecture**: Features like Kubernetes API integration, caching, metrics, forward, and rewrite are implemented as composable plugins.

## 3. How Service Discovery Works
1. When a Service is created, the Kubernetes API Server registers the Service name and assigned ClusterIP.
2. The CoreDNS Kubernetes plugin continuously watches the API Server for Service and Endpoint events.
3. CoreDNS maintains up-to-date DNS records mapping `<service>.<namespace>.svc.cluster.local` -> ClusterIP.

## 4. CoreDNS Configuration (Corefile)
CoreDNS is configured through a ConfigMap named `coredns` in `kube-system`:
```text
.:53 {
    errors
    health {
       lameduck 5s
    }
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
       pods insecure
       fallthrough in-addr.arpa ip6.arpa
       ttl 30
    }
    prometheus :9153
    forward . /etc/resolv.conf
    cache 30
    loop
    reload
    loadbalance
}
```

## 5. Troubleshooting DNS Issues
1. **Verify CoreDNS Pods**:
   `kubectl get pods -n kube-system -l k8s-app=kube-dns`
2. **Check CoreDNS Logs**:
   `kubectl logs -n kube-system -l k8s-app=kube-dns`
3. **Test DNS Resolution from a Pod**:
   `kubectl run dnsutils --image=tutum/dnsutils --rm -it --restart=Never -- nslookup kubernetes.default`
4. **Inspect Pod Resolv.conf**:
   `kubectl exec -it <pod-name> -- cat /etc/resolv.conf`
