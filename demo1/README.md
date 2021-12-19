

https://ubuntu.com/tutorials/install-a-local-kubernetes-with-microk8s#5-host-your-first-service-in-kubernetes


```bash
kubectl create namespace demo1
kubectl config set-context --current --namespace=demo1

kubectl create deployment microbot --image=dontrebootme/microbot:v1
kubectl scale deployment microbot --replicas=2
kubectl expose deployment microbot --type=NodePort --port=80 --name=microbot-service
```

Verification

```bash
kubectl get all
```
Get the node port and open browser `http://<your vm ip>:<nodeport>/`
