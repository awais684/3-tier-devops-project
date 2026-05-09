# Prometheus & Grafana Setup on EKS

## 1. Add Helm Repo
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

## 2. Install Kube-Prometheus Stack
Installs Prometheus, Grafana, Alertmanager, Node Exporter, and Kube State Metrics together.
```bash
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace
```

## 3. Verify Pods
```bash
kubectl get pods -n monitoring
```

```
kubectl patch svc monitoring-kube-prometheus-prometheus -n monitoring \
  -p '{"spec": {"type": "LoadBalancer"}}'
```
OR

## 4. Create Ingress for Grafana & Prometheus
Create `monitoring-ingress.yml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: monitoring-ingress
  namespace: monitoring
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  ingressClassName: alb
  rules:
  - host: grafana.techsubscribers.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: monitoring-grafana
            port:
              number: 80
  - host: prometheus.techsubscribers.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: monitoring-kube-prometheus-prometheus
            port:
              number: 9090
```
```bash
kubectl apply -f monitoring-ingress.yml
```

## 5. Get ALB URL
```bash
kubectl get ingress -n monitoring
```

## 6. Route 53 DNS Records
Create two A records in your hosted zone:

| Record Name | Type | Alias | Target |
|---|---|---|---|
| `grafana` | A | ON | monitoring ALB |
| `prometheus` | A | ON | monitoring ALB |

- Go to **Route 53 → techsubscribers.com → Create Record**
- Type: `A`
- Alias: ON
- Route traffic to: `Application and Classic Load Balancer`
- Region: `us-east-1`
- Select the monitoring ALB from the dropdown

```bash
# Get password
kubectl get secret monitoring-grafana -n monitoring \
  -o jsonpath="{.data.admin-password}" | base64 --decode

# Get username
kubectl get secret monitoring-grafana -n monitoring \
  -o jsonpath="{.data.admin-user}" | base64 --decode
```

## 8. Grafana Pre-built Dashboards
Once logged in, go to **Dashboards → Browse**. The following dashboards come pre-configured:
- Kubernetes Cluster Monitoring
- Node Exporter
- Pod Metrics
- Namespace Metrics

