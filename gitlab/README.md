
# Installation

Gitlab-ee
* https://docs.gitlab.com/ee/install/docker.html
* https://docs.gitlab.com/charts/installation/

Download [value.yaml](value.yaml) 

```bash
kubectl create namespace gitlab
kubectl config set-context --current --namespace=gitlab
microk8s enable helm3
microk8s.helm3 repo add gitlab https://charts.gitlab.io/
microk8s.helm3 repo update

microk8s.helm3 -n gitlab upgrade \
--install gitlab gitlab/gitlab  \
--timeout 600s  \
--set certmanager-issuer.email=kubesummit2021@canonical.com \
-f value.yaml

```

Have to wait maybe 10-15 min, take a coffee here ~ 

# Get Root Password

```bash
kubectl -n gitlab get secret gitlab-gitlab-initial-root-password -o jsonpath='{.data.password}' | base64 -d && echo
```

Use root/<password> login later

# Handle self-signed certificate issue

* Import the CA into browser
* Import the CA into system 

```bash
kubectl get secret -n gitlab gitlab-wildcard-tls-ca -o jsonpath="{['data']['cfssl_ca']}" | base64 --decode > gitlab-ca.crt
sudo cp gitlab-ca.crt /usr/local/share/ca-certificates
sudo update-ca-certificates -f
microk8s stop && microk8s start
```

# Verification

Check if the endpoint available

```bash
kubectl describe svc gitlab-webservice-default
kubectl describe ing gitlab-webservice-default
```

It should works when access from VM. For your laptop, you can type "thisisunsafe" to bypass the TLS checking in chrome.

```bash
curl https://gitlab.example.com/
```

* Check login 
* Check the runner






