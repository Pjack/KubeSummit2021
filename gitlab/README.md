
# Installation

Gitlab-ee
* https://docs.gitlab.com/ee/install/docker.html
* https://docs.gitlab.com/charts/installation/

Download [value.yaml](value.yaml) 

```bash
microk8s enable helm3
microk8s.helm3 repo add gitlab https://charts.gitlab.io/
microk8s.helm3 repo update

microk8s.helm3 -n gitlab upgrade \
--install gitlab gitlab/gitlab  \
--timeout 600s  \
--set certmanager-issuer.email=kubesummit2021@canonical.com \
-f value.yaml

```

Get Root Password

```bash
kubectl -n gitlab get secret gitlab-gitlab-initial-root-password -o jsonpath='{.data.password}' | base64 -d && echo
```

Handle self-signed certificate issue
* Import the CA into browser
* Import the CA into system 

```bash
kubectl get secret -n gitlab gitlab-wildcard-tls -o jsonpath="{['data']['cfssl_ca']}" | base64 --decode > gitlab-ca.crt
sudo cp gitlab-ca.crt /usr/local/share/ca-certificates
sudo update-ca-certificates -f
microk8s stop && microk8s start
```

# Try It!
It should works when access from VM. For your laptop, you can type "thisisunsafe" to bypass the TLS checking in chrome.

```bash
curl https://gitlab.example.com/
```
