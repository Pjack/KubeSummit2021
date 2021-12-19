# KubeSummit2021

## Preliminary

If you want to join the workshop this afternoon, it's better to finish this part before the workshop. The network bandwidth may not enough for image downloading at the same time. 

Microk8s can be installed in Linux/MacOS/Windows directly. But the instrustion below is only verified on Ubuntu 20.04. VM is much convenience during the workshop.

### Platform for virtual machine

Multipass: https://multipass.run/docs

* MacOS
  * Homebrew Installation if need
  ``` bash
  /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
  ```
* Windows
* Linux

Other reference for `how to use multipass`
* https://computingforgeeks.com/run-ubuntu-virtual-machines-on-linux-macos-using-multipass/
* https://www.how2shout.com/linux/how-to-install-mutliple-ubuntu-vms-using-multipass-on-ubunut-20-04/

### Ubuntu LTS (20.04)

Please use **lower** case for the VM's name. It will be used by hostname and upper case may cause some problem in k8s.

```bash
# Gitlab server needs 4G~5G ram here
multipass launch -c 4 -m 6G -d 32G -n workshop
multipass list
multipass shell workshop
ping 8.8.8.8
```

Troubleshooting
if public network is not available, it may be the default FORWARD policy is DROP in iptable. Setup the iptables on your laptop.

```bash
sudo iptables -L
sudo apt-get install iptables-persistent
sudo iptables -P FORWARD ACCEPT
sudo iptables -L
```

## MicroK8s Installation

https://ubuntu.com/tutorials/install-a-local-kubernetes-with-microk8s

### Basic

Installation

```bash
sudo snap install microk8s --classic --channel=1.23/stable
microk8s status
sudo usermod -a -G microk8s $USER
mkdir ~/.kube
sudo chown -f -R $USER ~/.kube
echo "alias kubectl='microk8s kubectl'" >> ~/.bash_aliases
echo 'source <(microk8s.kubectl completion bash)' >>~/.bashrc
```
logout and enter the shell again

Verification

```bash
microk8s status --wait-ready
kubectl get nodes
kubectl get services
kubectl cluster-info
```

### DNS (Only for Workshop)

Create a self-managed DNS that we can assign ip for self-managed gitlab later.

```bash
sudo apt update
sudo apt-get install -y dnsmasq
sudo sed -i 's/#interface=.*/interface=vxlan.calico/' /etc/dnsmasq.conf
sudo sed -i 's/#bind-interfaces/bind-interfaces/' /etc/dnsmasq.conf
sudo systemctl restart dnsmasq.service
```

### Turn on add-on

Get your host ip by `ip --brief address show vxlan.calico`

```bash
microk8s enable storage helm3
microk8s enable dns:<your host ip>
```

Put these entries in /etc/hosts that we will use it later
```text
<host ip> gitlab.example.com
<host ip> registry.example.com
<host ip> mimio.example.com
```

Verification
```
kubectl run -it --rm --restart=Never --image=infoblox/dnstools:latest dnstools
nslookup www.google.com
nslookup kubernetes.default.svc.cluster.local
nslookup gitlab.example.com
```
