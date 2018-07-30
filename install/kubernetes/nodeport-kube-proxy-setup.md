
# NodePort Service support in contrail

Contrail works with and depends on, kube-proxy component of the kubernetes software stack to implement support for NodePort Service feature in kubernetes.

With Docker version >= 1.13, kube-proxy needs the following configuration to get NodePort feature working seamlessly.

## Pre-requisites

A kubernetes master node with un-initialized kubernetes cluster i.e node where "kubeadm init" has not been executed.

If "kubeadm init" has already been issued, then the cluster needs to be torn down by executing "kubeadm reset" on ALL (master and compute) nodes of the cluster, before the below steps can be executed.

### Step 1. Create a yaml file with following config.

***NOTE: "10.32.0.0/12" here is the Pod network that the Contrail cluster was started with.***

Filename: kube-proxy-config.yaml   <-- Any name of your chosing.

```
apiVersion: kubeadm.k8s.io/v1alpha2
kind: MasterConfiguration
kubernetesVersion: v1.11.1
api:
  bindPort: 6443
kubeProxy:
  config:
    clusterCIDR: "10.32.0.0/12"
```

### Step 2. Instantiate the kubernetes cluster.

```
kubeadm init --config kube-proxy-config.yaml
```
