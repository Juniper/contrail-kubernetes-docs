
# Provisioning of Openshift cluster using openshift-ansible deployer 3.11

The following steps will install a standalone openshift cluster with Contrail as networking provider.

Provisioning of Openshift and Contrail is done through Ansible-playbooks.
Required topology is as shown below.

![Contrail Standalone Solution](/images/standalone-openshift-3.11.png)
* Note : vrouter is now installed on all nodes, the figure above does not have vrouter on master

### Steps :

* Setup environment(all nodes):
  
  * For centOS (origin installations) ( not supported ) : 
  
  * For Redhat (openshift-enterprise installations) : [click_here](/install/openshift/3.11/redhat/configurations.md)

* Get the files from the released and verified tar or git clone below : 

```shell
git clone https://github.com/Juniper/openshift-ansible.git -b release-3.11-contrail
```
* Note : Its recommend to get the code from the tar as the latest code may have behavior changes.


* For this setup am assuming one master one infra and one compute
```shell
master : server1 (10.84.11.11)

infra : server2 (10.84.11.22)

compute : server3 (10.84.11.33)
```

* Edit /etc/hosts to have all machines entry for eg(all nodes):

```shell
[root@server1]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.84.5.100 puppet
10.84.11.11 server1.contrail.juniper.net server1
10.84.11.22 server2.contrail.juniper.net server2
10.84.11.33 server3.contrail.juniper.net server3
```

* Setup passless ssh to ansible node itself and all nodes:

```shell
ssh-keygen -t rsa
ssh-copy-id root@10.84.11.11
ssh-copy-id root@10.84.11.22
ssh-copy-id root@10.84.11.33
```

### Run ansible playbook:
Before running make sure that you have edited inventory/ose-install file as shown below.

```shell 
ansible-playbook -i inventory/ose-install playbooks/prerequisites.yml
ansible-playbook -i inventory/ose-install playbooks/deploy_cluster.yml
```


### Sample ose-install file for non HA:

```yaml

[OSEv3:vars]

###########################################################################
### Ansible Vars
###########################################################################
#timeout=60

###########################################################################
### OpenShift Basic Vars
###########################################################################
openshift_deployment_type=openshift-enterprise
deployment_type=openshift-enterprise
containerized=false
openshift_disable_check=memory_availability,package_availability,disk_availability,package_version,docker_storage,docker_image_availability

# Default node selectors
openshift_hosted_infra_selector="node-role.kubernetes.io/infra=true"

# Redhat customer credentials
oreg_auth_user=<>
oreg_auth_password=<>
###########################################################################
### OpenShift Master Vars
###########################################################################

openshift_master_api_port=8443
openshift_master_console_port=8443

openshift_master_cluster_method=native
#openshift_master_cluster_hostname=ip-172-31-25-175.ap-southeast-1.compute.internal
#openshift_master_cluster_public_hostname=ec2-13-251-240-166.ap-southeast-1.compute.amazonaws.com
#openshift_master_default_subdomain=apps.ap-southeast-1.compute.amazonaws.com

# Set this line to enable NFS
openshift_enable_unsupported_configurations=True


#########################################################################
### Contrail Variables
########################################################################

contrail_version=5.0

contrail_container_tag=5.0.2-0.398-queens
contrail_registry_insecure=false
contrail_registry="hub.juniper.net/contrail-nightly"
contrail_registry_username=<>
contrail_registry_password=<>

#contrail_os_release=redhat7
#analyticsdb_min_diskgb=50
#configdb_min_diskgb=25
#aaa_mode=no-auth
#auth_mode=noauth
#CLOUD_ORCHESTRATOR=openshift
#LOG_LEVEL=SYS_NOTICE
#METADATA_PROXY_SECRET=contrail
#cloud_orchestrator=kubernetes
#metadata_proxy_secret=contrail
#log_level=SYS_NOTICE
#rabbitmq_node_port=5672
#zookeeper_analytics_port=2182
#zookeeper_port=2181
#zookeeper_ports=2888:3888
#zookeeper_analytics_ports=4888:5888
#vrouter_gateway=172.31.16.1
#vrouter_physical_interface=eth0
#kubernetes_api_secure_port=443
#nested_mode_contrail=false

# vip should be master
#api_vip="172.31.25.175"

service_subnets="172.30.0.0/16"
pod_subnets="10.128.0.0/14"

###########################################################################
### OpenShift Network Vars
###########################################################################

#os_sdn_network_plugin_name='redhat/openshift-ovs-networkpolicy'
openshift_use_openshift_sdn=false
#r_openshift_node_use_openshift_sdn=True
os_sdn_network_plugin_name='cni'
openshift_use_contrail=true
#openshift_use_calico=true


###########################################################################
### OpenShift Authentication Vars
###########################################################################

# htpasswd Authentication
openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider'}]


###########################################################################
### OpenShift Router and Registry Vars
###########################################################################

openshift_hosted_router_replicas=1

openshift_hosted_registry_replicas=1

openshift_hosted_registry_storage_kind=nfs
openshift_hosted_registry_storage_access_modes=['ReadWriteMany']
openshift_hosted_registry_storage_nfs_directory=/export
openshift_hosted_registry_storage_nfs_options='*(rw,root_squash)'
openshift_hosted_registry_storage_volume_name=registry
openshift_hosted_registry_storage_volume_size=10Gi
openshift_hosted_registry_pullthrough=true
openshift_hosted_registry_acceptschema2=true
openshift_hosted_registry_enforcequota=true
openshift_hosted_router_selector="node-role.kubernetes.io/infra=true"
openshift_hosted_registry_selector="node-role.kubernetes.io/infra=true"
###########################################################################
### OpenShift Service Catalog Vars
###########################################################################

openshift_enable_service_catalog=true

template_service_broker_install=true
openshift_template_service_broker_namespaces=['openshift']

ansible_service_broker_install=true
ansible_service_broker_local_registry_whitelist=['.*-apb$']

openshift_hosted_etcd_storage_kind=nfs
openshift_hosted_etcd_storage_nfs_options="*(rw,root_squash,sync,no_wdelay)"
openshift_hosted_etcd_storage_nfs_directory=/export
openshift_hosted_etcd_storage_labels={'storage': 'etcd-asb'}
openshift_hosted_etcd_storage_volume_name=etcd-asb
openshift_hosted_etcd_storage_access_modes=['ReadWriteOnce']
openshift_hosted_etcd_storage_volume_size=10G


###########################################################################
### OpenShift Metrics and Logging Vars
###########################################################################
# Enable cluster metrics
openshift_metrics_install_metrics=True

openshift_metrics_storage_kind=nfs
openshift_metrics_storage_access_modes=['ReadWriteOnce']
openshift_metrics_storage_nfs_directory=/export
openshift_metrics_storage_nfs_options='*(rw,root_squash)'
openshift_metrics_storage_volume_name=metrics
openshift_metrics_storage_volume_size=10Gi
openshift_metrics_storage_labels={'storage': 'metrics'}

openshift_metrics_cassandra_nodeselector={"node-role.kubernetes.io/infra":"true"}
openshift_metrics_hawkular_nodeselector={"node-role.kubernetes.io/master":"true"}
openshift_metrics_heapster_nodeselector={"node-role.kubernetes.io/master":"true"}

# Enable cluster logging
openshift_logging_install_logging=True

openshift_logging_storage_kind=nfs
openshift_logging_storage_access_modes=['ReadWriteOnce']
openshift_logging_storage_nfs_directory=/export
openshift_logging_storage_nfs_options='*(rw,root_squash)'
openshift_logging_storage_volume_name=logging
openshift_logging_storage_volume_size=10Gi
openshift_logging_storage_labels={'storage': 'logging'}

openshift_logging_es_cluster_size=1

openshift_logging_es_nodeselector={"node-role.kubernetes.io/infra":"true"}
openshift_logging_kibana_nodeselector={"node-role.kubernetes.io/infra":"true"}
openshift_logging_curator_nodeselector={"node-role.kubernetes.io/infra":"true"}


###########################################################################
### OpenShift Prometheus Vars
###########################################################################

## Add Prometheus Metrics:
openshift_hosted_prometheus_deploy=true
openshift_prometheus_node_selector={"node-role.kubernetes.io/infra":"true"}
openshift_prometheus_namespace=openshift-metrics

# Prometheus
openshift_prometheus_storage_kind=nfs
openshift_prometheus_storage_access_modes=['ReadWriteOnce']
openshift_prometheus_storage_nfs_directory=/export
openshift_prometheus_storage_nfs_options='*(rw,root_squash)'
openshift_prometheus_storage_volume_name=prometheus
openshift_prometheus_storage_volume_size=10Gi
openshift_prometheus_storage_labels={'storage': 'prometheus'}
openshift_prometheus_storage_type='pvc'

# For prometheus-alertmanager
openshift_prometheus_alertmanager_storage_kind=nfs
openshift_prometheus_alertmanager_storage_access_modes=['ReadWriteOnce']
openshift_prometheus_alertmanager_storage_nfs_directory=/export
openshift_prometheus_alertmanager_storage_nfs_options='*(rw,root_squash)'
openshift_prometheus_alertmanager_storage_volume_name=prometheus-alertmanager
openshift_prometheus_alertmanager_storage_volume_size=10Gi
openshift_prometheus_alertmanager_storage_labels={'storage': 'prometheus-alertmanager'}
openshift_prometheus_alertmanager_storage_type='pvc'

# For prometheus-alertbuffer
openshift_prometheus_alertbuffer_storage_kind=nfs
openshift_prometheus_alertbuffer_storage_access_modes=['ReadWriteOnce']
openshift_prometheus_alertbuffer_storage_nfs_directory=/export
openshift_prometheus_alertbuffer_storage_nfs_options='*(rw,root_squash)'
openshift_prometheus_alertbuffer_storage_volume_name=prometheus-alertbuffer
openshift_prometheus_alertbuffer_storage_volume_size=10Gi
openshift_prometheus_alertbuffer_storage_labels={'storage': 'prometheus-alertbuffer'}
openshift_prometheus_alertbuffer_storage_type='pvc'


###########################################################################
### OpenShift Hosts
###########################################################################
[OSEv3:children]
masters
etcd
nodes
nfs

[masters]
a6s41node1

[etcd]
a6s41node1

[nodes]
a6s41node1 openshift_node_group_name='node-config-master'
a6s41node2 openshift_node_group_name='node-config-compute'
a6s4node1 openshift_node_group_name='node-config-infra'

[nfs]
a6s41node2

```
### Sample ose-install for Nested mode :

```yaml
[OSEv3:vars]

###########################################################################
### Ansible Vars
###########################################################################
#timeout=60

###########################################################################
### OpenShift Basic Vars
###########################################################################
openshift_deployment_type=openshift-enterprise
deployment_type=openshift-enterprise
containerized=false
openshift_disable_check=docker_image_availability,memory_availability,package_availability,disk_availability,package_version,docker_storage

# Default node selectors
openshift_hosted_infra_selector="node-role.kubernetes.io/infra=true"

oreg_auth_user=<>
oreg_auth_password=<>
###########################################################################
### OpenShift Master Vars
###########################################################################

openshift_master_api_port=8443
openshift_master_console_port=8443

openshift_master_cluster_method=native
#openshift_master_cluster_hostname=ip-172-31-25-175.ap-southeast-1.compute.internal
#openshift_master_cluster_public_hostname=ec2-13-251-240-166.ap-southeast-1.compute.amazonaws.com
#openshift_master_default_subdomain=apps.ap-southeast-1.compute.amazonaws.com

# Set this line to enable NFS
openshift_enable_unsupported_configurations=True


#########################################################################
### Contrail Variables
########################################################################

contrail_version=5.0

contrail_container_tag=5.1.0-0.511-rhel-queens
contrail_registry_insecure=false
contrail_registry="hub.juniper.net/contrail-nightly"
contrail_registry_username=<>
contrail_registry_password=<>

nested_mode_contrail=true
auth_mode=keystone
# contrail nested masters string seperated by space
contrail_nested_masters_ip="10.84.13.51"
keystone_auth_host=10.84.13.51
keystone_auth_admin_tenant=admin
keystone_auth_admin_user=admin
keystone_auth_admin_password=MAYffWrX7ZpPrV2AMAa9zAUvG
keystone_auth_admin_port=35357
keystone_auth_url_version=/v3
#k8s_nested_vrouter_vip is a service IP for the running node which we configured above
k8s_nested_vrouter_vip=10.10.10.5
#k8s_vip is kubernetes api server ip
k8s_vip=192.168.100.3
#cluster_network is the one which vm network belongs to
cluster_network="{'domain': 'default-domain', 'project': 'admin', 'name': 'openshift-vn'}"

#contrail_os_release=redhat7
#analyticsdb_min_diskgb=50
#configdb_min_diskgb=25
#aaa_mode=no-auth
#auth_mode=noauth
#CLOUD_ORCHESTRATOR=openshift
#LOG_LEVEL=SYS_NOTICE
#METADATA_PROXY_SECRET=contrail
#cloud_orchestrator=kubernetes
#metadata_proxy_secret=contrail
#log_level=SYS_NOTICE
#rabbitmq_node_port=5673
#zookeeper_analytics_port=2182
#zookeeper_port=2181
#zookeeper_ports=2888:3888
#zookeeper_analytics_ports=4888:5888
#vrouter_gateway=172.31.16.1
#vrouter_physical_interface=eth0
#kubernetes_api_secure_port=443
#nested_mode_contrail=false

# vip should be master
#api_vip="172.31.25.175"

service_subnets="172.30.0.0/16"
pod_subnets="10.128.0.0/14"

###########################################################################
### OpenShift Network Vars
###########################################################################

#os_sdn_network_plugin_name='redhat/openshift-ovs-networkpolicy'
openshift_use_openshift_sdn=false
#r_openshift_node_use_openshift_sdn=True
os_sdn_network_plugin_name='cni'
openshift_use_contrail=true
#openshift_use_calico=true


###########################################################################
### OpenShift Authentication Vars
###########################################################################

# htpasswd Authentication
openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider'}]


###########################################################################
### OpenShift Router and Registry Vars
###########################################################################

openshift_hosted_router_replicas=1

openshift_hosted_registry_replicas=1

openshift_hosted_registry_storage_kind=nfs
openshift_hosted_registry_storage_access_modes=['ReadWriteMany']
openshift_hosted_registry_storage_nfs_directory=/export
openshift_hosted_registry_storage_nfs_options='*(rw,root_squash)'
openshift_hosted_registry_storage_volume_name=registry
openshift_hosted_registry_storage_volume_size=10Gi
openshift_hosted_registry_pullthrough=true
openshift_hosted_registry_acceptschema2=true
openshift_hosted_registry_enforcequota=true
openshift_hosted_router_selector="node-role.kubernetes.io/infra=true"
openshift_hosted_registry_selector="node-role.kubernetes.io/infra=true"
###########################################################################
### OpenShift Service Catalog Vars
###########################################################################

openshift_enable_service_catalog=true

template_service_broker_install=true
openshift_template_service_broker_namespaces=['openshift']

ansible_service_broker_install=true
#ansible_service_broker_local_registry_whitelist=['.*-apb$']

openshift_hosted_etcd_storage_kind=nfs
openshift_hosted_etcd_storage_nfs_options="*(rw,root_squash,sync,no_wdelay)"
#openshift_hosted_etcd_storage_nfs_options="*(rw,root_squash)"
openshift_hosted_etcd_storage_nfs_directory=/export
openshift_hosted_etcd_storage_labels={'storage': 'etcd-asb'}
openshift_hosted_etcd_storage_volume_name=etcd-asb
openshift_hosted_etcd_storage_access_modes=['ReadWriteOnce']
openshift_hosted_etcd_storage_volume_size=10G


###########################################################################
### OpenShift Metrics and Logging Vars
###########################################################################
# Enable cluster metrics
openshift_metrics_install_metrics=True
#openshift_metrics_cassandra_storage_type=pv
#openshift_metrics_hawkular_hostname=hawkular-metrics.example.com

openshift_metrics_storage_kind=nfs
openshift_metrics_storage_access_modes=['ReadWriteOnce']
openshift_metrics_storage_nfs_directory=/export
openshift_metrics_storage_nfs_options='*(rw,root_squash)'
openshift_metrics_storage_volume_name=metrics
openshift_metrics_storage_volume_size=10Gi
openshift_metrics_storage_labels={'storage': 'metrics'}

openshift_metrics_cassandra_nodeselector={"node-role.kubernetes.io/infra":"true"}
openshift_metrics_hawkular_nodeselector={"node-role.kubernetes.io/infra":"true"}
openshift_metrics_heapster_nodeselector={"node-role.kubernetes.io/infra":"true"}

# Enable cluster logging
openshift_logging_install_logging=True

openshift_logging_storage_kind=nfs
openshift_logging_storage_access_modes=['ReadWriteOnce']
openshift_logging_storage_nfs_directory=/export
openshift_logging_storage_nfs_options='*(rw,root_squash)'
openshift_logging_storage_volume_name=logging
openshift_logging_storage_volume_size=10Gi
openshift_logging_storage_labels={'storage': 'logging'}

openshift_logging_es_cluster_size=1

openshift_logging_es_nodeselector={"node-role.kubernetes.io/infra":"true"}
openshift_logging_kibana_nodeselector={"node-role.kubernetes.io/infra":"true"}
openshift_logging_curator_nodeselector={"node-role.kubernetes.io/infra":"true"}


###########################################################################
### OpenShift Prometheus Vars
###########################################################################

## Add Prometheus Metrics:
openshift_hosted_prometheus_deploy=true
openshift_prometheus_node_selector={"node-role.kubernetes.io/infra":"true"}
openshift_prometheus_namespace=openshift-metrics

# Prometheus
openshift_prometheus_storage_kind=nfs
openshift_prometheus_storage_access_modes=['ReadWriteOnce']
openshift_prometheus_storage_nfs_directory=/export
openshift_prometheus_storage_nfs_options='*(rw,root_squash)'
openshift_prometheus_storage_volume_name=prometheus
openshift_prometheus_storage_volume_size=10Gi
openshift_prometheus_storage_labels={'storage': 'prometheus'}
openshift_prometheus_storage_type='pvc'

# For prometheus-alertmanager
openshift_prometheus_alertmanager_storage_kind=nfs
openshift_prometheus_alertmanager_storage_access_modes=['ReadWriteOnce']
openshift_prometheus_alertmanager_storage_nfs_directory=/export
openshift_prometheus_alertmanager_storage_nfs_options='*(rw,root_squash)'
openshift_prometheus_alertmanager_storage_volume_name=prometheus-alertmanager
openshift_prometheus_alertmanager_storage_volume_size=10Gi
openshift_prometheus_alertmanager_storage_labels={'storage': 'prometheus-alertmanager'}
openshift_prometheus_alertmanager_storage_type='pvc'

# For prometheus-alertbuffer
openshift_prometheus_alertbuffer_storage_kind=nfs
openshift_prometheus_alertbuffer_storage_access_modes=['ReadWriteOnce']
openshift_prometheus_alertbuffer_storage_nfs_directory=/export
openshift_prometheus_alertbuffer_storage_nfs_options='*(rw,root_squash)'
openshift_prometheus_alertbuffer_storage_volume_name=prometheus-alertbuffer
openshift_prometheus_alertbuffer_storage_volume_size=10Gi
openshift_prometheus_alertbuffer_storage_labels={'storage': 'prometheus-alertbuffer'}
openshift_prometheus_alertbuffer_storage_type='pvc'


###########################################################################
### OpenShift Hosts
###########################################################################
[OSEv3:children]
masters
etcd
nodes
nfs

[masters]
a6s41node1

[etcd]
a6s41node1

[nodes]
a6s41node1 openshift_node_group_name='node-config-master'
a6s41node2 openshift_node_group_name='node-config-compute'
a6s4node1 openshift_node_group_name='node-config-infra'

[nfs]
a6s41node2 openshift_hostname=a6s41node2

```

### Sample ose-install file for HA:
```yaml
[OSEv3:vars]

###########################################################################
### Ansible Vars
###########################################################################
#timeout=60

###########################################################################
### OpenShift Basic Vars
###########################################################################
openshift_deployment_type=openshift-enterprise
deployment_type=openshift-enterprise
containerized=false
openshift_disable_check=memory_availability,package_availability,disk_availability,package_version,docker_storage

# Default node selectors
openshift_hosted_infra_selector="node-role.kubernetes.io/infra=true"

oreg_auth_user=<>
oreg_auth_password=<>
###########################################################################
### OpenShift Master Vars
###########################################################################

openshift_master_api_port=8443
openshift_master_console_port=8443

openshift_master_cluster_method=native
openshift_master_cluster_hostname=lb
openshift_master_cluster_public_hostname=lb
#openshift_master_default_subdomain=

# Set this line to enable NFS
openshift_enable_unsupported_configurations=True


#########################################################################
### Contrail Variables
########################################################################

contrail_version=5.0
openshift_use_contrail=true

contrail_container_tag=5.0.2-0.398-queens
contrail_registry_insecure=false
contrail_registry="hub.juniper.net/contrail-nightly"
contrail_registry_username=<>
contrail_registry_password=<>


#contrail_os_release=redhat7
#analyticsdb_min_diskgb=50
#configdb_min_diskgb=25
#aaa_mode=no-auth
#auth_mode=noauth
#CLOUD_ORCHESTRATOR=openshift
#LOG_LEVEL=SYS_NOTICE
#METADATA_PROXY_SECRET=contrail
#cloud_orchestrator=kubernetes
#metadata_proxy_secret=contrail
#log_level=SYS_NOTICE
#rabbitmq_node_port=5672
#zookeeper_analytics_port=2182
#zookeeper_port=2181
#zookeeper_ports=2888:3888
#zookeeper_analytics_ports=4888:5888
#vrouter_gateway=172.31.16.1
#vrouter_physical_interface=eth0
#kubernetes_api_secure_port=443
#nested_mode_contrail=false

# vip should be master
#api_vip="172.31.25.175"

service_subnets="172.30.0.0/16"
pod_subnets="10.128.0.0/14"


###########################################################################
### OpenShift Network Vars
###########################################################################

#os_sdn_network_plugin_name='redhat/openshift-ovs-networkpolicy'
openshift_use_openshift_sdn=false
r_openshift_node_use_openshift_sdn=True
os_sdn_network_plugin_name='cni'

###########################################################################
### OpenShift Authentication Vars
###########################################################################

# htpasswd Authentication
openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider'}]


###########################################################################
### OpenShift Router and Registry Vars
###########################################################################

openshift_hosted_router_replicas=1

openshift_hosted_registry_replicas=1

openshift_hosted_registry_storage_kind=nfs
openshift_hosted_registry_storage_access_modes=['ReadWriteMany']
openshift_hosted_registry_storage_nfs_directory=/export
openshift_hosted_registry_storage_nfs_options='*(rw,root_squash)'
openshift_hosted_registry_storage_volume_name=registry
openshift_hosted_registry_storage_volume_size=10Gi
openshift_hosted_registry_pullthrough=true
openshift_hosted_registry_acceptschema2=true
openshift_hosted_registry_enforcequota=true
openshift_hosted_router_selector="node-role.kubernetes.io/infra=true"
openshift_hosted_registry_selector="node-role.kubernetes.io/infra=true"
###########################################################################
### OpenShift Service Catalog Vars
###########################################################################

openshift_enable_service_catalog=true

template_service_broker_install=true
openshift_template_service_broker_namespaces=['openshift']

ansible_service_broker_install=true
ansible_service_broker_local_registry_whitelist=['.*-apb$']

openshift_hosted_etcd_storage_kind=nfs
openshift_hosted_etcd_storage_nfs_options="*(rw,root_squash,sync,no_wdelay)"
openshift_hosted_etcd_storage_nfs_directory=/export
openshift_hosted_etcd_storage_labels={'storage': 'etcd-asb'}
openshift_hosted_etcd_storage_volume_name=etcd-asb
openshift_hosted_etcd_storage_access_modes=['ReadWriteOnce']
openshift_hosted_etcd_storage_volume_size=10G


###########################################################################
### OpenShift Metrics and Logging Vars
###########################################################################
# Enable cluster metrics
openshift_metrics_install_metrics=True

openshift_metrics_storage_kind=nfs
openshift_metrics_storage_access_modes=['ReadWriteOnce']
openshift_metrics_storage_nfs_directory=/export
openshift_metrics_storage_nfs_options='*(rw,root_squash)'
openshift_metrics_storage_volume_name=metrics
openshift_metrics_storage_volume_size=10Gi
openshift_metrics_storage_labels={'storage': 'metrics'}

openshift_metrics_cassandra_nodeselector={"node-role.kubernetes.io/infra":"true"}
openshift_metrics_hawkular_nodeselector={"node-role.kubernetes.io/infra":"true"}
openshift_metrics_heapster_nodeselector={"node-role.kubernetes.io/infra":"true"}

# Enable cluster logging
openshift_logging_install_logging=True

openshift_logging_storage_kind=nfs
openshift_logging_storage_access_modes=['ReadWriteOnce']
openshift_logging_storage_nfs_directory=/export
openshift_logging_storage_nfs_options='*(rw,root_squash)'
openshift_logging_storage_volume_name=logging
openshift_logging_storage_volume_size=10Gi
openshift_logging_storage_labels={'storage': 'logging'}

openshift_logging_kibana_hostname=kibana.apps.ap-southeast-1.compute.amazonaws.com
openshift_logging_es_cluster_size=1

openshift_logging_es_nodeselector={"node-role.kubernetes.io/infra":"true"}
openshift_logging_kibana_nodeselector={"node-role.kubernetes.io/infra":"true"}
openshift_logging_curator_nodeselector={"node-role.kubernetes.io/infra":"true"}


###########################################################################
### OpenShift Prometheus Vars
###########################################################################

## Add Prometheus Metrics:
openshift_hosted_prometheus_deploy=true
openshift_prometheus_node_selector={"node-role.kubernetes.io/infra":"true"}
openshift_prometheus_namespace=openshift-metrics

# Prometheus
openshift_prometheus_storage_kind=nfs
openshift_prometheus_storage_access_modes=['ReadWriteOnce']
openshift_prometheus_storage_nfs_directory=/export
openshift_prometheus_storage_nfs_options='*(rw,root_squash)'
openshift_prometheus_storage_volume_name=prometheus
openshift_prometheus_storage_volume_size=10Gi
openshift_prometheus_storage_labels={'storage': 'prometheus'}
openshift_prometheus_storage_type='pvc'
# For prometheus-alertmanager
openshift_prometheus_alertmanager_storage_kind=nfs
openshift_prometheus_alertmanager_storage_access_modes=['ReadWriteOnce']
openshift_prometheus_alertmanager_storage_nfs_directory=/export
openshift_prometheus_alertmanager_storage_nfs_options='*(rw,root_squash)'
openshift_prometheus_alertmanager_storage_volume_name=prometheus-alertmanager
openshift_prometheus_alertmanager_storage_volume_size=10Gi
openshift_prometheus_alertmanager_storage_labels={'storage': 'prometheus-alertmanager'}
openshift_prometheus_alertmanager_storage_type='pvc'
# For prometheus-alertbuffer
openshift_prometheus_alertbuffer_storage_kind=nfs
openshift_prometheus_alertbuffer_storage_access_modes=['ReadWriteOnce']
openshift_prometheus_alertbuffer_storage_nfs_directory=/export
openshift_prometheus_alertbuffer_storage_nfs_options='*(rw,root_squash)'
openshift_prometheus_alertbuffer_storage_volume_name=prometheus-alertbuffer
openshift_prometheus_alertbuffer_storage_volume_size=10Gi
openshift_prometheus_alertbuffer_storage_labels={'storage': 'prometheus-alertbuffer'}
openshift_prometheus_alertbuffer_storage_type='pvc'


###########################################################################
### OpenShift Hosts
###########################################################################
[OSEv3:children]
masters
etcd
nodes
nfs
lb
openshift_ca

[masters]
kube-master-0-e4c1bd8c1f8740e18aca00c95fcb5936
kube-master-1-e4c1bd8c1f8740e18aca00c95fcb5936
kube-master-2-e4c1bd8c1f8740e18aca00c95fcb5936

[etcd]
kube-master-0-e4c1bd8c1f8740e18aca00c95fcb5936
kube-master-1-e4c1bd8c1f8740e18aca00c95fcb5936
kube-master-2-e4c1bd8c1f8740e18aca00c95fcb5936

[nodes]
kube-master-0-e4c1bd8c1f8740e18aca00c95fcb5936 openshift_node_group_name='node-config-master'
kube-master-1-e4c1bd8c1f8740e18aca00c95fcb5936 openshift_node_group_name='node-config-master'
kube-master-2-e4c1bd8c1f8740e18aca00c95fcb5936 openshift_node_group_name='node-config-master'
controller-0-e4c1bd8c1f8740e18aca00c95fcb5936 openshift_node_group_name='node-config-infra'
controller-1-e4c1bd8c1f8740e18aca00c95fcb5936 openshift_node_group_name='node-config-infra'
controller-2-e4c1bd8c1f8740e18aca00c95fcb5936 openshift_node_group_name='node-config-infra'
compute-0-e4c1bd8c1f8740e18aca00c95fcb5936 openshift_node_group_name='node-config-compute'

[nfs]
compute-1-e4c1bd8c1f8740e18aca00c95fcb5936

[lb]
load-balancer-0-e4c1bd8c1f8740e18aca00c95fcb5936

[openshift_ca]
kube-master-0-e4c1bd8c1f8740e18aca00c95fcb5936
kube-master-1-e4c1bd8c1f8740e18aca00c95fcb5936
kube-master-2-e4c1bd8c1f8740e18aca00c95fcb5936

```

### Issues
* if there is a java error do, yum install java-1.8.0-openjdk-devel.x86_64 and rerun deploy_cluster
* if the service_catalog is not passing but cluster is up fine, check /etc/resolv.conf whether it has cluster.local
  in its search line, and nameserver as host ip
* If you see pods being evicted or pending, check ur disk usage
* If you see tcp timeout issue but the master is up fine and you can reach it, check network connectivity speeds (usually vms         have this problem )

### Note:
* use "oc adm manage-node --selector=region=infra --schedulable=false" to make infra nodes non schedulable


### Make OpenShift web console working
Create a password for admin user to login to the UI from master node
```
(master-node)# htpasswd /etc/origin/master/htpasswd admin
```
**If you are using a lb, you manually need to copy the htpasswd file into all your masters

Assign cluster-admin role to admin user
```
(master-node)# oc adm policy add-cluster-role-to-user cluster-admin admin
(master-node)# oc login -u admin
```

### Accessing web console
* Go to browser and type the entire fqdn name of your master node / lb node, followed by :8443/console
```
https://<your host name from your ose-install inventory>:8443/console
```
* use username/password created above to login into webconsole
* Note : your dns should resolve for the above hostname for access, else modify your /etc/hosts file to route
  to above host

