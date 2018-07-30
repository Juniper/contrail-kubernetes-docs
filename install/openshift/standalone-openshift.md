### Reimage all your servers with : 

```shell
/cs-shared/server-manager/client/server-manager reimage --server_id server1 centos-7.4
```

### Setup environment(all nodes):

```shell
yum install vim git wget -y && wget -O /tmp/epel-release-latest-7.noarch.rpm https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm && rpm -ivh /tmp/epel-release-latest-7.noarch.rpm && yum update -y && yum install python-pip -y && pip install ansible==2.5.2
```

### Clone ansible repo (ansible node): 

```shell
git clone https://github.com/Juniper/openshift-ansible.git -b release-3.7-contrail
```

### For this setup am assuming one master one slave

master : server1 (10.84.11.11)

slave : server2 (10.84.11.22)

### Edit /etc/hosts to have all machines entry for eg(all nodes):

```shell
[root@server1]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.84.5.100 puppet
10.84.11.11 server1.contrail.juniper.net server1
10.84.11.22 server2.contrail.juniper.net server2
20.1.1.1    server1.contrail.juniper.net server1
20.1.1.2    server2.contrail.juniper.net server2
```

### Setup passless ssh to ansible node itself and all nodes:

```shell
ssh-keygen -t rsa
ssh-copy-id root@10.84.11.11
ssh-copy-id root@10.84.11.22
```

### Run ansible playbook:
Before running make sure that you have edited inventory/byo/ose-install file as shown below

```shell 
ansible-playbook -i inventory/byo/ose-install inventory/byo/ose-prerequisites.yml
ansible-playbook -i inventory/byo/ose-install playbooks/byo/openshift_facts.yml
ansible-playbook -i inventory/byo/ose-install playbooks/byo/config.yml
```



### Note (temporary fixes):

If you see any of below couple of different TASK errors :

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
ansible-playbook -i inventory/byo/ose-install playbooks/byo/config.yml

```


### Sample ose-install file:

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
deployment_type=origin
openshift_release=v3.7
openshift_pkg_version=-3.7.1-2.el7
openshift_repos_enable_testing=true
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
openshift_disable_check=disk_availability,package_version,docker_storage
# Below are Contrail variables. Comment them out if you don't want to install Contrail through ansible-playbook

openshift_use_contrail=true
contrail_container_tag=master-centos7-newton-bld-17
contrail_registry=docker.io/opencontrailnightly
contrail_nodes=10.84.11.11
vrouter_physical_interface={interface-name(eg eth0)}
contrail_vip=10.84.11.11

[masters]
10.84.11.11 openshift_hostname=server1

[etcd]
10.84.11.11 openshift_hostname=server1

[nodes]
10.84.11.11 openshift_hostname=server1
10.84.11.22 openshift_hostname=server2

[openshift_ca]
10.84.11.11 openshift_hostname=server1
```
