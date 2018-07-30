
# Disclaimer

***This guide is meant to be a helpful reference to install Kubernetes. It is NOT intended to be an authoritative guide for Kubernetes install. It is merely steps that we use everyday and happens to work for us.***

Please refer to canonical documentation from Kubernetes community  [here](https://kubernetes.io/docs/setup/)

# Installing Kuberntes on Ubuntu-16.04 hosts

## Installing Kubernetes on Master node

1. Prepare the node by running the following pre-requisites
```
swapoff -a
sudo apt-get update
sudo apt-get install -y curl software-properties-common apt-transport-https
```

2. Install Docker
```
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install -y docker-ce # Install docker
sudo service docker status  # Verify that docker service is running
```

3. Add Kubernetes repo
```
sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
```
4. Install Kubernetes components
```
sudo apt-get install -y kubectl kubelet kubeadm
```
5. Config kubeadm if NodePort service is needed.(OPTIONAL)

   [Kube-proxy config](install/nodeport-kube-proxy-setup.md)

6. Create K8s cluster

  If kubeadm was configured for nodeport (via step 5):
```
kubeadm init --config config.yaml
```
  else
```
kubeadm init
```
7. Once "kubeadm init" completes, save the "join" command that will be printed on the shell, to a file of your choice. This will be needed to add new nodes to your cluster.

```
example:
"kubeadm join 192.168.1.3:6443 --token 0smq4g.7pmg2jqc8arl1uz7 --discovery-token-ca-cert-hash sha256:d92ac0785b1435666d726f4bc54fde58693f87cf91371d9fd553da4a40813650"
```
8. Run the following commands to initially kubernetes command line
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

9. Disable firewalld
```
sysctl -w net.bridge.bridge-nf-call-iptables=1
systemctl stop firewalld; systemctl disable firewalld
```

10. Disable networking on Master if desired. (OPTIONAL)

  Disabling networking on the master will result not workloads not being scheduled on the master node.

   For kubeadm version >= 1.11

  ```
  a. sudo vi /var/lib/kubelet/kubeadm-flags.env

  b. Remove the following 3 arguments from KUBELET_KUBEADM_ARGS variable:
          --cni-bin-dir=/opt/cni/bin
          --cni-conf-dir=/etc/cni/net.d
          --network-plugin=cni

  c. Restart kubelet Service

      systemctl enable kubelet && systemctl start kubelet
  ```

  For kubeadm version < 1.11

  ```
  a. sudo vi /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

  b. Comment out

  "#Environment="KUBELET_NETWORK_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin"
  sudo systemctl daemon-reload;sudo service kubelet restart

  c. Restart kubelet Service

     systemctl enable kubelet && systemctl start kubelet
  ```

## Installing Kubernetes on Compute (k8s slave/minion) nodes

  1. Prepare the node by running the following pre-requisites
  ```
  swapoff -a
  sudo apt-get update
  sudo apt-get install -y curl software-properties-common apt-transport-https
  ```

  2. Install Docker
  ```
  sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
  sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
  sudo apt-get update
  sudo apt-get install -y docker-ce # Install docker
  sudo service docker status  # Verify that docker service is running
  ```

  3. Add Kubernetes repo
  ```
  sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
  sudo echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
  sudo apt-get update
  ```
  4. Install Kubernetes components
  ```
  sudo apt-get install -y kubectl kubelet
  ```
  5. Disable firewalld
  ```
  sysctl -w net.bridge.bridge-nf-call-iptables=1
  systemctl stop firewalld; systemctl disable firewalld
  ```

  6. Join the Master node

  Copy paste the "join" command you saved from Step-7 of instructions for installation of K8s on master.
  ```
  "kubeadm join 192.168.1.3:6443 --token 0smq4g.7pmg2jqc8arl1uz7 --discovery-token-ca-cert-hash sha256:d92ac0785b1435666d726f4bc54fde58693f87cf91371d9fd553da4a40813650"
  ```

# Installing Kuberntes on Centos hosts

## Installing Kubernetes on Master node

**(Centos 7.4 - 3.10.0-862.3.2 kernel)**

1. Prepare the node by running the following pre-requisites
  ```
sudo setenforce 0
swapoff -a
```
2. Install Docker

  ```
sudo yum install -y docker
systemctl enable docker.service;service docker start
```

3. Add Kubernetes repo
  ```
cat << EOF >> /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```
4. Install Kubernetes components
  ```
yum update -y; yum install -y kubelet kubeadm
```

5. Config kubeadm if NodePort service is needed.(OPTIONAL)

   [Kube-proxy config](install/nodeport-kube-proxy-setup.md)

6. Create K8s cluster

  If kubeadm was configured for nodeport (via step 5):
  ```
kubeadm init --config config.yaml
```
  else
```
kubeadm init
```
7. Once "kubeadm init" completes, save the "join" command that will be printed on the shell, to a file of your choice. This will be needed to add new nodes to your cluster.

  ```
example:
"kubeadm join 192.168.1.3:6443 --token 0smq4g.7pmg2jqc8arl1uz7 --discovery-token-ca-cert-hash sha256:d92ac0785b1435666d726f4bc54fde58693f87cf91371d9fd553da4a40813650"
```
8. Run the following commands to initially kubernetes command line
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
9. Disable firewalld
```
sysctl -w net.bridge.bridge-nf-call-iptables=1
systemctl stop firewalld; systemctl disable firewalld
```
10. Disable networking on Master if desired. (OPTIONAL)

  Disabling networking on the master will result not workloads not being scheduled on the master node.

   For kubeadm version >= 1.11

  ```
  a. sudo vi /var/lib/kubelet/kubeadm-flags.env

  b. Remove the following 3 arguments from KUBELET_KUBEADM_ARGS variable:
          --cni-bin-dir=/opt/cni/bin
          --cni-conf-dir=/etc/cni/net.d
          --network-plugin=cni

  c. Restart kubelet Service

      systemctl enable kubelet && systemctl start kubelet
  ```

  For kubeadm version < 1.11

  ```
  a. sudo vi /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

  b. Comment out

  "#Environment="KUBELET_NETWORK_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin"
  sudo systemctl daemon-reload;sudo service kubelet restart

  c. Restart kubelet Service

     systemctl enable kubelet && systemctl start kubelet
  ```

## Installing Kubernetes on Compute (k8s slave/minion) nodes
**(Centos 7.4 - 3.10.0-862.3.2 kernel)**

1. Prepare the node by running the following pre-requisites
  ```
sudo setenforce 0
swapoff -a
```

2. Install Docker
  ```
sudo yum install -y docker
systemctl enable docker.service;service docker start
```
3. Add Kubernetes repo
  ```
cat << EOF >> /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```
4. Install Kubernetes components
  ```
yum update -y; yum install -y kubelet kubeadm
```
5. Disable firewalld
  ```
sysctl -w net.bridge.bridge-nf-call-iptables=1
systemctl stop firewalld; systemctl disable firewalld
```
6. Join the Master node
Copy paste the "join" command you saved from Step-9 of instructions for installation of K8s on master.
```
"kubeadm join 192.168.1.3:6443 --token 0smq4g.7pmg2jqc8arl1uz7 --discovery-token-ca-cert-hash sha256:d92ac0785b1435666d726f4bc54fde58693f87cf91371d9fd553da4a40813650"
```
