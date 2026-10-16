# Fully Qualified Domain Names (FQDN) in Kubernetes

## 1. What is an FQDN?
A Fully Qualified Domain Name (FQDN) is the complete domain name for a specific computer, or host, on the internet or within a private network. In Kubernetes, every Service and Pod gets an internal DNS name that resolves within the cluster.

## 2. Kubernetes DNS Naming Convention
The standard syntax for a Kubernetes Service FQDN is:
`<service-name>.<namespace>.svc.cluster.local`

- `<service-name>`: Name of the Kubernetes Service resource.
- `<namespace>`: The namespace where the Service is deployed (e.g., `default`, `kube-system`).
- `svc`: Indicates that this object is a Service.
- `cluster.local`: The default cluster domain suffix.

## 3. Namespace-based DNS Resolution
- **Same Namespace**: A Pod can reach a service using just its short name:
  `curl http://my-service:80`
- **Across Namespaces**: When communicating across namespaces, the namespace must be included:
  `curl http://my-service.other-namespace:80`
- **Full FQDN**: Always unambiguous across the entire cluster:
  `curl http://my-service.other-namespace.svc.cluster.local:80`

## 4. Pod-to-Service Communication
When a client Pod initiates a connection to `my-service`:
1. The container queries `/etc/resolv.conf`, which points to CoreDNS (`nameserver 10.96.0.10`).
2. CoreDNS resolves `my-service.default.svc.cluster.local` to the Service's virtual **ClusterIP**.
3. `kube-proxy` (via iptables or IPVS) intercepts the traffic directed to the ClusterIP and load-balances packets across the actual backend Pod IPs matching the Service selector.

## 5. Examples of Kubernetes FQDNs
| Service / Pod | Namespace | Full FQDN |
| :--- | :--- | :--- |
| `web-service` | `default` | `web-service.default.svc.cluster.local` |
| `auth-db` | `database` | `auth-db.database.svc.cluster.local` |
| `redis-master` | `cache` | `redis-master.cache.svc.cluster.local` |
| Headless Pod 0 | `prod` | `stateful-0.stateful-svc.prod.svc.cluster.local` |
