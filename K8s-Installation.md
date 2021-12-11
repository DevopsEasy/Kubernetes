## Kubernetes Installation
* We are using AWS to create Servers
* Create two t2.medium machines one is master & another is node.
* Install Kuberenets using kubeadm [Refer Here](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
* Before installing kubernetes we need to install Container Runtime.
* We are using docker as a container runtime.
* Docker Installation Steps [Refer Here](https://get.docker.com/)

```
# login as root user
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
sudo usermod -aG docker ubuntu
# exit & relogin
```

* After Installaling Docker Execute This Steps [Refer Here](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#docker)

```
sudo mkdir /etc/docker
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

sudo systemctl enable docker
sudo systemctl daemon-reload
sudo systemctl restart docker
```
### Install Kubelet, Kubeadm & Kubectl

* Update the apt package index and install packages needed to use the Kubernetes apt repository:

```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```

* Download the Google Cloud public signing key:

```
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```

* Add the Kubernetes apt repository:

```
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
* Update apt package index, install kubelet, kubeadm and kubectl, and pin their version:

```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```
* Repeat steps for both master and node

## Kubeadm Master Setup
* Login as root user

```
kubeadm init
# login as normal user (ex: ubuntu)
mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
# Kubenetes Master will not be in ready state
* For that we need to install network plugin
* We are using weavenet
```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```
* After that login into Node

```
kubeadm join 172.31.18.219:6443 --token fqe1yr.w75nigm3yjynbg8z \
        --discovery-token-ca-cert-hash sha256:e805ba076065ea7ba8255f7ceba8e04afe01891587d134020719203101075bc8

```


