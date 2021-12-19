
# Enable K8s dashboard

https://microk8s.io/docs/addon-dashboard

Enable Dashboard and Shared Nginx Ingress

```bash
microk8s enable dashboard ingress
```

# Add Ingress for the dashboard

Create dashboard-ing.yaml, please remember to change the <yourvmip>.

```yamlhttps://dashboard-10.18.24.178.nip.io/
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: dashboard
  namespace: kube-system
  annotations:
    # use the shared ingress-nginx
    kubernetes.io/ingress.class: public
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  # https://kubernetes.io/docs/concepts/services-networking/ingress/
  # https://kubernetes.github.io/ingress-nginx/user-guide/tls/
  rules:
  - host: dashboard-<yourvmip>.nip.io
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:                                                                                                                
          service:
            name: kubernetes-dashboard
            port:
              number: 443
```

```bash
kubectl apply -f dashboard-ing.yaml
token=$(microk8s kubectl -n kube-system get secret | grep default-token | cut -d " " -f1)
kubectl -n kube-system describe secret $token
```

Open browser e.g. `https://dashboard-10.18.24.178.nip.io/`

Note: type `thisisunsafe` to bypass the self-signed certificate or import it into chrome browser

# Better after 1.23

```bash
microk8s enable dashboard-ingress 
```