
# Installation

Gitlab-ee
* https://docs.gitlab.com/ee/install/docker.html
* https://docs.gitlab.com/charts/installation/

Download [value.yaml](value.yaml) 

Note: This value.yaml is designed for gitlab server using self-signed certificate.
For other case, please refer https://docs.gitlab.com/charts/installation/tls.html

```bash
kubectl create namespace gitlab
kubectl config set-context --current --namespace=gitlab
microk8s enable helm3 ingress
microk8s.helm3 repo add gitlab https://charts.gitlab.io/
microk8s.helm3 repo update

wget https://raw.githubusercontent.com/Pjack/KubeSummit2021/main/gitlab/value.yaml
microk8s.helm3 -n gitlab upgrade \
--install gitlab gitlab/gitlab  \
--timeout 600s  \
--set certmanager-issuer.email=kubesummit2021@canonical.com \
-f value.yaml

# The total memory size set by gitlab is over 8G which is too large to most laptop
# shrink it to smaller size

kubectl set resources deployment gitlab-sidekiq-all-in-1-v2 --requests cpu=100m,memory=256Mi
kubectl set resources deployment gitlab-webservice-default --requests cpu=100m,memory=256Mi
kubectl scale --replicas=0 deployment gitlab-sidekiq-all-in-1-v2 
kubectl scale --replicas=1 deployment gitlab-sidekiq-all-in-1-v2 
kubectl scale --replicas=0 deployment gitlab-webservice-default
kubectl scale --replicas=2 deployment gitlab-webservice-default
```

Have to wait maybe 10-15 min, take a coffee here ~ 

# Get Root Password

```bash
kubectl -n gitlab get secret gitlab-gitlab-initial-root-password -o jsonpath='{.data.password}' | base64 -d && echo
```
Use root and the password login later

# Handle self-signed certificate

* Import the CA into browser
* Import the CA into system 

```bash
kubectl get secret -n gitlab gitlab-wildcard-tls-ca -o jsonpath="{['data']['cfssl_ca']}" | base64 --decode > gitlab-ca.crt
sudo cp gitlab-ca.crt /usr/local/share/ca-certificates
sudo update-ca-certificates -f
```

Troubleshooting: may need to restart microk8s to reload the certificate
```bash
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

Modify /etc/hosts on the laptop , use the ip of ens3 on VM
```text
10.18.24.155 gitlab.example.com
```
* Check login 
* Check the runner

# Demo Project

Import the project from [simple-agent-k](https://gitlab.com/pjack.chen/simple-agent-k), Try CI/CD with your micro cloud

# KAS & Agent
 
https://docs.gitlab.com/ee/user/clusters/agent/install/index.html
https://about.gitlab.com/blog/2020/09/22/introducing-the-gitlab-kubernetes-agent/

You may encounter this issue when the gitlab server using self-signed certificate.  
  
https://docs.gitlab.com/ee/user/clusters/agent/#error-while-dialing-failed-to-websocket-dial-failed-to-send-handshake-request   

> It’s not possible to set the grpc scheme due to the issue It is not possible to configure KAS to work with grpc without directly editing GitLab KAS deployment. To use grpc while the issue is in progress, directly edit the deployment with the kubectl edit deployment gitlab-kas command, and change --listen-websocket=true to --listen-websocket=false. After running that command, you should be able to use grpc://gitlab-kas.<YOUR-NAMESPACE>:8150.


You need to disable the web-socket at k8s agent server. (Already setup in [value.yaml](value.yaml))

And then change the kas-address to internal grpc  
  
```bash
kubectl edit deployment -n gitlab-kubernetes-agent gitlab-agent
```

Change the kas-address as `grpc://gitlab-kas.gitlab.svc.cluster.local:8150`
  




