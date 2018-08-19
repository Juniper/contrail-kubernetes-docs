Nested mode is when Contrail provides networking for a Kubernetes cluster that is provisioned on an Contail-Openstack cluster. Contrail components are shared between the two clusters.

![Contrail Standalone Solution](/images/nested-kubernetes.png)

# __Prerequisites__

Please ensure that the following prerequisites are met, for a successful provisioning of Nested Contrail-Kubernetes cluster.

1. Installed and running Contrail Openstack cluster.
   This cluster should be based on Contrail 5.0 release 

2. Installed and running Kubernetes cluster on Virtual Machines created on Contrail Openstack cluster.
   User is free to follow any installation method of their choice. 

  2.a. Update /etc/hosts file on your kubernetes master node with entries for each node of your cluster.
```
      Example: 
          If you kubernetes cluster is made up of three nodes:
          master1 (IP: x.x.x.x), minion1 (IP: y.y.y.y) and minion2 (IP: z.z.z.z)

          /etc/hosts on the kubernetes master node should have the following entries:

          x.x.x.x master1
          y.y.y.y minion1
          z.z.z.z minion2
```
3. If Contrail container images are stored in private/secure docker registry, a kubernetes secret should be created and referenced during creation of single yaml, with credentials of the private docker registry.

```
   kubectl create secret docker-registry <name> --docker-server=<registry> --docker-username=<username> --docker-password=<password> --docker-email=<email> -n <namespace>

   <name>      - name of the secret
   <registry>  - example: hub.juniper.net/contrail
   <username>  - registry user name
   <password>  - registry passcode
   <email>     - registered email of this registry account
   <namespace> - kubernetes namespace where this secret is to be created. 
                 This should be the namespace where you intend to create Contrail pods.

   ```

# __Provision__
  
  Provisioning a Nested Kubernetes Cluster is a three step process:

  ***1. Configure network connectivity to Contrail Config/Data plane functions.***

  ***2. Generate single yaml file to create Contrail-k8s cluster.***

  ***3. Instantiate Contrail-k8s cluster.***

## Configure network connectivity to Contrail Config/Data plane functions

In a nested deployement, one kube-manager instance will be provisioned in each overlay cluster. Kube-manager is essentially a part of Contrail config function. This necessitates the need for kube-manager running in overlay to have network reachability to Contrail Config functions of the underlay Openstack cluster.

Network connectivity for the following Contrail Config functions are required:

|  |
| --- |
| Contrail Config     |
| Contrail Analytics  |
| Contrail Msg Queue  |
| Contrail VNC DB     |
| Keystone            |

In addition to Config connectivity, CNI for the Kubernetes cluster needs network reachability to the Vrouter Agent of its Compute node.

Network connectivity for the following Data plane function is required:

|  |
| --- |
| Vrouter   |

We can use Link Local Service feature or a combination of Link Local + Fabric SNAT feature of Contrail to provide IP reachability to/from the overlay kubernetes cluster config/data compoenents and to corresponding config/data compoenents of the underlay openstack cluster.

### Option 1: Fabric SNAT + Link Local (Preferred)

Step 1: Enable Fabric SNAT on the Virtual Network of the VM's

Fabric SNAT feature should be enabled on the Virtual Network of the Virtual Machine's on which Kubernetes Master and Minions are running.

Step 2: Create one Link Local Service so CNI to communicate with its Vrouter Agent.

To configure a Link Local Service, we need a Service IP and Fabric IP. Fabric IP is the node IP on which the vrouter agent of the minion is running. Service IP(along with port number) is used by data plane to identify the fabric ip/node. Service IP is required to be a unique and unused IP in the entire openstack cluster. 

***NOTE: The user is responsible to configure these Link Local Services via Contrail GUI.***

The following are the Link Local Service is required:

| Contrail Process | Service IP  | Service Port | Fabric IP | Fabric Port |
| --- | --- | --- | --- | --- |
| VRouter             | < Service IP for the running node > | 9091 | 127.0.0.1 | 9091 |

NOTE: Fabric IP is 127.0.0.1, as our intent is to make CNI talk to Vrouter on its underlay node.

####Example:

The following link-local services should be created:

| LL Service Name | Service IP  | Service Port | Fabric IP | Fabric Port |
| --- | --- | --- | --- | --- |
| K8s-cni-to-agent | 10.10.10.5 | 9091 | 127.0.0.1 | 9091 |

NOTE: Here 10.10.10.5 is the Service IP that was chosen by user. This can be any unused
IP in the cluster. This IP is primarily used to identify link local traffic and has no
other signifance.


### Option 2: Link Local Only

To configure a Link Local Service, we need a Service IP and Fabric IP. Fabric IP is the node IP on which the contrail processes are running on. Service IP(along with port number) is used by data plane to identify the fabric ip/node. Service IP is required to be a unique and unused IP in the entire openstack cluster. **For each node of the openstack cluster, one service IP should be identified.**

***NOTE: The user is responsible to configure these Link Local Services via Contrail GUI.***

The following are the Link Local Services are required:

| Contrail Process | Service IP  | Service Port | Fabric IP | Fabric Port |
| --- | --- | --- | --- | --- |
| Contrail Config     | < Service IP for the running node > | 8082 | < Node IP of running node > | 8082 |
| Contrail Analytics  | < Service IP for the running node > | 8086 | < Node IP of running node > | 8086 |
| Contrail Msg Queue  | < Service IP for the running node > | 5673 | < Node IP of running node > | 5673 |
| Contrail VNC DB     | < Service IP for the running node > | 9161 | < Node IP of running node > | 9161 |
| Keystone            | < Service IP for the running node > | 35357 | < Node IP of running node > | 35357 |
| VRouter             | < Service IP for the running node > | 9091 | 127.0.0.1 | 9091 |

####Example:

Lets assume the following hypothetical Openstack Cluster where:
```
Contrail Config : 192.168.1.100
Contrail Analytics : 192.168.1.100, 192.168.1.101
Contrail Msg Queue : 192.168.1.100
Contrail VNC DB : 192.168.1.100, 192.168.1.101, 192.168.1.102
Keystone: 192.168.1.200
Vrouter: 192.168.1.300, 192.168.1.400, 192.168.1.500
```
This cluster is made of 7 nodes. We will allocate 7 unused IP's for these nodes:
```
192.168.1.100  --> 10.10.10.1
192.168.1.101  --> 10.10.10.2
192.168.1.102  --> 10.10.10.3
192.168.1.200  --> 10.10.10.4
192.168.1.300  --> 10.10.10.5
192.168.1.400  --> 10.10.10.6
192.168.1.500  --> 10.10.10.7
```
The following link-local services should be created:

| LL Service Name | Service IP  | Service Port | Fabric IP | Fabric Port |
| --- | --- | --- | --- | --- |
| Contrail Config      | 10.10.10.1 | 8082 | 192.168.1.100 | 8082 |
| Contrail Analytics 1 | 10.10.10.1 | 8086 | 192.168.1.100 | 8086 |
| Contrail Analytics 2 | 10.10.10.2 | 8086 | 192.168.1.101 | 8086 |
| Contrail Msg Queue   | 10.10.10.1 | 5673 | 192.168.1.100 | 5673 |
| Contrail VNC DB 1    | 10.10.10.1 | 9161 | 192.168.1.100 | 9161 |
| Contrail VNC DB 2    | 10.10.10.2 | 9161 | 192.168.1.101 | 9161 |
| Contrail VNC DB 3    | 10.10.10.3 | 9161 | 192.168.1.102 | 9161 |
| Keystone             | 10.10.10.4 | 35357 | 192.168.1.200| 35357 |
| VRouter-192.168.1.300 | 10.10.10.5 | 9091 | 127.0.0.1 | 9091 |
| VRouter-192.168.1.400 | 10.10.10.6 | 9091 | 127.0.0.1 | 9091 |
| VRouter-192.168.1.500 | 10.10.10.7 | 9091 | 127.0.0.1 | 9091 |

## Generate yaml file

Contrail components will be installed on the Kubernetes cluster as pods.
The config to create these Pods in K8s is encoded in a yaml file.

This file can be generated as follows:

Step 1: 

Clone contrail-container-build repo in any server of your choice.
```
git clone https://github.com/Juniper/contrail-container-builder.git
```

Step 2:

Populate common.env file (located in the top directory of the cloned contrail-container-builder repo) with info corresponding to your cluster and environment.

For you reference, please find a sample common.env file with required bare minimum configurations here:

https://github.com/Juniper/contrail-container-builder/blob/master/kubernetes/sample_config_files/common.env.sample.nested_mode

**NOTE:
   If Contrail container images are stored in private/secure docker registry, a kubernetes secret should have be created, as 
   documented in pre-requesites. Populate the variable KUBERNETES_SECRET_CONTRAIL_REPO=< secret-name > with the name of the 
   generated secret,in common.env file.**

Step 3:

Generate the yaml file as following in your shell:
```
cd <your contrail-container-build-repo>/kubernetes/manifests

./resolve-manifest.sh contrail-kubernetes-nested.yaml  > nested-contrail.yml
```

Step 4:

Copy over the output (or file) generated from Step(3) to the master node in your kubernetes cluster.

## Instantiate Contrail-k8s cluster

Create a contrail components as pods on the kubernetes cluster, as follows:

```
kubectl apply -f nested-contrail.yml
```

You will see the following pods running in the "kube-system" namespace:

contrail-kube-manager-xxxxxx          --> This is the manager that acts as conduit between kubernetes and openstack clusters.

contrail-kubernetes-cni-agent-xxxxx   --> This installs and configures Contrail CNI on kubernetes nodes.

```
root@k8s:~# kubectl get pods -n kube-system
NAME                                  READY     STATUS    RESTARTS   AGE
contrail-kube-manager-lcjbc           1/1       Running   0          3d
contrail-kubernetes-cni-agent-w8shc   1/1       Running   0          3d
```

***Now you are ready to created workloads on the Kubernetes cluster***

