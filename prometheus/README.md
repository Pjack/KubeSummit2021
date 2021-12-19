# Addon in Microk8s

Turn on the addon in micriok8s. 

```bash
microk8s enable ingress prometheus
kubectl get svc -n monitoring
```

Export the service to host
* Solution 1:
```bash
kubectl port-forward -n monitoring service/prometheus-k8s --address 0.0.0.0 9090:9090
kubectl port-forward -n monitoring service/grafana --address 0.0.0.0 3000:3000
```
* Solution 2: Change service type from ClusterIP to NodePort
* Solution 3: Create ingress in front of the service

There are default template in grafana

Grafana password: admin/admin

# Managed by Juju

https://juju.is/docs/lma2/install/microk8s

```bash
snap 
juju bootstrap microk8s mk8s
juju add-model lma
juju switch lma
juju deploy lma-light --channel=edge --trust
juju status  
juju run-action grafana-k8s/0 get-admin-password --wait
```

Reference
* https://github.com/charmed-lma/charm-k8s-grafana
* https://discourse.ubuntu.com/t/logging-monitoring-and-alerting/19151
* https://juju.is/blog/lma-2-modern-observability-with-juju-by-charming-the-grafana-ecosystem
* https://ubuntu.com/blog/monitoring-at-the-edge-with-microk8s* 


