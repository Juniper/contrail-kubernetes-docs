
# Provisioning of Openshift cluster using openshift-ansible deployer 3.9

The following steps will install a standalone openshift cluster with Contrail as networking provider.

Provisioning of Openshift and Contrail is done through Ansible-playbooks.
Required topology is as shown below.

![Contrail Standalone Solution](/images/standalone-openshift-3.9.png)

### Steps :

* Setup environment(all nodes):
  
  * For centOS (origin installations) : [click_here](/install/openshift/3.9/centos/configurations.md)
  
  * For Redhat (openshift-enterprise installations) : [click_here](/install/openshift/3.9/redhat/configurations.md)
  
* Needs supported ansible version
  
  ```shell
  yum install -y python-pip
  pip install ansible==2.5.2
  ```
* Get the files from the released and verified tar or git clone below : 

```shell
git clone https://github.com/Juniper/openshift-ansible.git -b release-3.9-contrail
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

### Note (temporary fixes):

1. If you see any of below couple of different TASK errors :

```shell 
TASK [contrail_node : Label master nodes with opencontrail.org/controller=true] ******************************************************************************************************
Tuesday 20 March 2018  12:22:35 -0700 (0:00:00.672)       0:16:36.099 *********
failed: [10.84.11.22 -> 10.84.11.11] (item=10.84.11.11) => {"changed": true, "cmd": ["oc", "label", "nodes", "server1", "opencontrail.org/controller=true", "--overwrite=true"], "delta": "0:00:00.212255", "end": "2018-03-20 12:22:35.756727", "item": "10.84.11.11", "msg": "non-zero return code", "rc": 1, "start": "2018-03-20 12:22:35.544472", "stderr": "Error from server (NotFound): nodes \"server1\" not found", "stderr_lines": ["Error from server (NotFound): nodes \"server1\" not found"], "stdout": "", "stdout_lines": []}

(or)
TASK openshift_node :  restart node
RUNNING HANDLER [openshift_node : restart node] ***********************************************************************************************************************************************************
Wednesday 21 March 2018  14:19:48 -0700 (0:00:00.086)       0:14:48.981 ******* 
FAILED - RETRYING: restart node (3 retries left).
FAILED - RETRYING: restart node (3 retries left).
FAILED - RETRYING: restart node (3 retries left).
FAILED - RETRYING: restart node (2 retries left).
FAILED - RETRYING: restart node (2 retries left).
FAILED - RETRYING: restart node (2 retries left).
FAILED - RETRYING: restart node (1 retries left).
FAILED - RETRYING: restart node (1 retries left).
FAILED - RETRYING: restart node (1 retries left).
fatal: [10.87.36.11]: FAILED! => {"attempts": 3, "changed": false, "msg": "Unable to restart service origin-node: Job for origin-node.service failed because the control process exited with error code. See \"systemctl status origin-node.service\" and \"journalctl -xe\" for details.\n"}
fatal: [10.87.36.10]: FAILED! => {"attempts": 3, "changed": false, "msg": "Unable to restart service origin-node: Job for origin-node.service failed because the control process exited with error code. See \"systemctl status origin-node.service\" and \"journalctl -xe\" for details.\n"}
fatal: [10.87.36.12]: FAILED! => {"attempts": 3, "changed": false, "msg": "Unable to restart service origin-node: Job for origin-node.service failed because the control process exited with error code. See \"systemctl status origin-node.service\" and \"journalctl -xe\" for details.\n"}
```

Fix:

```shell
touch /etc/origin/node/resolv.conf

rerun :
ansible-playbook -i inventory/ose-install playbooks/deploy_cluster.yml

```
2. If you see docker image pull errors, and your registry is a insecure registry. set openshift_docker_insecure_registries=<your registry> in the ose-install and rerun prerequisites play. 

```

### Run ansible playbook:
Before running make sure that you have edited inventory/ose-install file as shown below.

```shell 
ansible-playbook -i inventory/ose-install playbooks/prerequisites.yml
ansible-playbook -i inventory/ose-install playbooks/deploy_cluster.yml
```


### Sample ose-install file for non HA:

```yaml
[OSEv3:children]
masters
nodes
etcd
openshift_ca

[OSEv3:vars]
ansible_ssh_user=root
ansible_become=yes
debug_level=2
deployment_type=origin #openshift-enterprise for Redhat
openshift_release=v3.9
#openshift_repos_enable_testing=true
containerized=false
openshift_install_examples=true
openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider', 'filename': '/etc/origin/master/htpasswd'}]
osm_cluster_network_cidr=10.32.0.0/12
openshift_portal_net=10.96.0.0/12
openshift_use_dnsmasq=true
openshift_clock_enabled=true
openshift_hosted_manage_registry=false
openshift_hosted_manage_router=false
openshift_enable_service_catalog=false
openshift_use_openshift_sdn=false
os_sdn_network_plugin_name='cni'
openshift_disable_check=memory_availability,package_availability,disk_availability,package_version,docker_storage
openshift_docker_insecure_registries=opencontrailnightly
openshift_web_console_install=false
#openshift_web_console_nodeselector={'region':'infra'}

openshift_web_console_contrail_install=true
openshift_use_contrail=true
nested_mode_contrail=false
contrail_version=5.0
contrail_container_tag=ocata-5.0-156
contrail_registry=opencontrailnightly
# Username /Password for private Docker regiteries
#contrail_registry_username=test
#contrail_registry_password=test
# Below option presides over contrail masters if set
#vrouter_physical_interface=ens160
#docker_version=1.13.1
ntpserver=10.1.1.1 # a proper ntpserver is required for contrail.

# Contrail_vars
# below variables are used by contrail kubemanager to configure the cluster,
# you can configure all options below. All values are defaults and can be modified.

#kubernetes_api_server=10.84.13.52         # in our case this is the master, which is default
#kubernetes_api_port=8080               
#kubernetes_api_secure_port=8443
#cluster_name=myk8s                  
#cluster_project={}
#cluster_network={}
#pod_subnets=10.32.0.0/12
#ip_fabric_subnets=10.64.0.0/12
#service_subnets=10.96.0.0/12
#ip_fabric_forwarding=false
#ip_fabric_snat=false
#public_fip_pool={}
#vnc_endpoint_ip=20.1.1.1
#vnc_endpoint_port=8082

[masters]
10.84.13.52 openshift_hostname=openshift-master

[etcd]
10.84.13.52 openshift_hostname=openshift-master

[nodes]
10.84.13.52 openshift_hostname=openshift-master
10.84.13.53 openshift_hostname=openshift-compute
10.84.13.54 openshift_hostname=openshift-infra openshift_node_labels="{'region': 'infra'}"

[openshift_ca]
10.84.13.52 openshift_hostname=openshift-master
```

### Sample ose-install file for HA:
```yaml
[OSEv3:children]
masters
nodes
etcd
lb
openshift_ca

[OSEv3:vars]
ansible_ssh_user=root
ansible_become=yes
debug_level=2
deployment_type=openshift-enterprise
openshift_release=v3.9
openshift_repos_enable_testing=true
containerized=false
openshift_install_examples=true
openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider', 'filename': '/etc/origin/master/htpasswd'}]
osm_cluster_network_cidr=10.32.0.0/12
openshift_portal_net=10.96.0.0/12
openshift_use_dnsmasq=true
openshift_clock_enabled=true
openshift_enable_service_catalog=false
openshift_use_openshift_sdn=false
os_sdn_network_plugin_name='cni'
openshift_disable_check=disk_availability,package_version,docker_storage
openshift_docker_insecure_registries=ci-repo.englab.juniper.net:5010
openshift_web_console_install=false
openshift_web_console_contrail_install=true
openshift_web_console_nodeselector={'region':'infra'}
openshift_hosted_manage_registry=true
openshift_hosted_registry_selector="region=infra"
openshift_hosted_manage_router=true
openshift_hosted_router_selector="region=infra"
ntpserver=<your ntpserver>


# Openshift HA
openshift_master_cluster_method=native
openshift_master_cluster_hostname=lb
openshift_master_cluster_public_hostname=lb


# Below are Contrail variables. Comment them out if you don't want to install Contrail through ansible-playbook
contrail_version=5.0
openshift_use_contrail=true
contrail_registry=hub.juniper.net/contrail
contrail_registry_username=<username>
contrail_registry_password=<password>
contrail_container_tag=<image-tag>
#vrouter_physical_interface=eth0

[masters]
10.0.0.13 openshift_hostname=master1
10.0.0.4 openshift_hostname=master2
10.0.0.5 openshift_hostname=master3

[lb]
10.0.0.6 openshift_hostname=lb

[etcd]
10.0.0.13 openshift_hostname=master1
10.0.0.4 openshift_hostname=master2
10.0.0.5 openshift_hostname=master3

[nodes]
10.0.0.13 openshift_hostname=master1
10.0.0.4 openshift_hostname=master2
10.0.0.5 openshift_hostname=master3
10.0.0.7 openshift_hostname=slave1
10.0.0.10 openshift_hostname=slave2
10.0.0.9 openshift_hostname=infra1 openshift_node_labels="{'region': 'infra'}"
10.0.0.11 openshift_hostname=infra2 openshift_node_labels="{'region': 'infra'}"
10.0.0.8 openshift_hostname=infra3 openshift_node_labels="{'region': 'infra'}"

[openshift_ca]
10.0.0.13 openshift_hostname=master1
10.0.0.4 openshift_hostname=master2
10.0.0.5 openshift_hostname=master3
```


### Note:
* dnsmasq on master needs to be restarted after installation if dns is not working as expected.
* use "oc adm manage-node --selector=region=infra --schedulable=false" to make infra nodes non schedulable


### Make OpenShift web console working
* We will be installing our customized web console which will run
on infra nodes.
* To do this disable the openshift web console and enable contrail's webconsole, add below in the ose-install
```
openshift_web_console_install=false
openshift_web_console_contrail_install=true
```

After above step

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

