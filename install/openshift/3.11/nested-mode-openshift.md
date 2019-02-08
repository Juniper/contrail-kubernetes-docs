Nested mode is when Contrail provides networking for a Openshift cluster that is provisioned on an Contail-Openstack cluster. Contrail components are shared between the two clusters.

# __Prerequisites__

Please ensure that the following prerequisites are met, for a successful provisioning of Nested Contrail-Openshift cluster.

- Installed and running Contrail Openstack cluster.
   This cluster should be based on Contrail 5.x release 


# __Provision__
  
  Provisioning a Nested Openshift Cluster is a two step process:

  ***1. Create link-local services in the Contrail-Openstack cluster.***

  ***2. Install openshift using openshift-ansible.***


## Create link-local services

A nested Openshift cluster is managed by the same contrail control processes that manage the underlying openstack cluster. Towards this goal, the nested Openshift cluster needs ip reachability to the contrail control processes. Since the Openshift cluster is actually an overlay on the openstack cluster, we use Link Local Service feature or a combination of Link Local + Fabric SNAT feature of Contrail to provide IP reachability to/from the overly Openshift cluster and openstack cluster. 

### Option 1: Fabric SNAT + Link Local (Preferred)

Step 1: Enable Fabric SNAT on the Virtual Network of the VM's

Fabric SNAT feature should be enabled on the Virtual Network of the Virtual Machine's on which Openshift Master and Nodes are running.

Step 2: Create one Link Local Service for CNI to communicate with its Vrouter

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
| K8s-cni-to-agent    | < Service IP for the running node > | 9091 | 127.0.0.1 | 9091 |

####Example:

Lets assume the following hypothetical Openstack Cluster where:
```
Contrail Config : 192.168.1.100
Contrail Analytics : 192.168.1.100, 192.168.1.101
Contrail Msg Queue : 192.168.1.100
Contrail VNC DB : 192.168.1.100, 192.168.1.101, 192.168.1.102
Keystone: 192.168.1.200
Vrouter: 192.168.1.201, 192.168.1.202, 192.168.1.203
```
This cluster is made of 7 nodes. We will allocate 7 unused IP's for these nodes:
```
192.168.1.100  --> 10.10.10.1
192.168.1.101  --> 10.10.10.2
192.168.1.102  --> 10.10.10.3
192.168.1.200  --> 10.10.10.4
192.168.1.201/192.168.1.202/192.168.1.203  --> 10.10.10.5 
   NOTE: One Service IP will be enough to represent all VRouter nodes.
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
| K8s-cni-to-agent | 10.10.10.5 | 9091 | 127.0.0.1 | 9091 |


## Install openshift using openshift-ansible

Now you can follow [stand-alone wiki](https://github.com/Juniper/contrail-kubernetes-docs/blob/master/install/openshift/3.9/standalone-openshift.md)
and add following details to your ose-install file 
```
#Nested mode vars
nested_mode_contrail=true
rabbitmq_node_port=5673
contrail_nested_masters_ip="1.1.1.1 2.2.2.2 3.3.3.3"
auth_mode=keystone
keystone_auth_host=<w.x.y.z>        <--- This should be the IP where Keystone service is running.
keystone_auth_admin_tenant=admin
keystone_auth_admin_user=admin
keystone_auth_admin_password=MAYffWrX7ZpPrV2AMAa9zAUvG     <-- Keystone admin password.
keystone_auth_admin_port=35357
keystone_auth_url_version=/v3
#k8s_nested_vrouter_vip is a service IP for the running node which we configured above
k8s_nested_vrouter_vip=10.10.10.5   <-- Service IP configured for CNI to Agent communication.(K8s-cni-to-agent in above examples)
#k8s_vip is kubernetes api server ip
k8s_vip=<W.X.Y.Z>                   <-- IP of the Openshift Master Node.
#cluster_network is the one which vm network belongs to
cluster_network="{'domain': 'default-domain', 'project': 'admin', 'name': 'net1'}" <-- FQName of the Virtual Network where Virtual Machines are running. There are the VM's in which Openshift cluster is being installed in nested mode.
```
