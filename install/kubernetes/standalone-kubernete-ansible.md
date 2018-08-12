# Provisioning of Kubernetes Cluster Using contrail-ansible-deployer

The following steps will install a standalone Kubernetes cluster with Contrail as networking provider.

Provisioning of K8s and Contrail is done through Ansible-playbooks.

![Contrail Standalone Solution](/images/standalone-kubernetes.png)

### Step 1. Re-image the node to CentOS 7.4.

**Linux kernel version 3.10.0-862.3.2**

   Contrail forwarding uses a kernel module to provide high throughput, low latency networking.

   The latest kernel module is compiled against 3.10.0-862.3.2 kernel.

### Step 2.	Install the necessary utilities.
```
yum -y install epel-release git ansible net-tools
```

### Step 3.	Clone the contrail-ansible-deployer repo.
```
git clone https://github.com/Juniper/contrail-ansible-deployer.git
cd contrail-ansible-deployer
```

### Step 4.	Edit the config/instances.yaml and enter the necessary values. 

For example, see the following sample file for a one node compute and a one node controller installation.
```
provider_config:
  bms:
   ssh_pwd: Password
   ssh_user: root
   ssh_public_key: /root/.ssh/id_rsa.pub
   ssh_private_key: /root/.ssh/id_rsa
   domainsuffix: local
instances:
  bms1:
   provider: bms
   roles:            # Optional.  If roles is not defined, all below roles will be created
      config_database:         # Optional.
      config:                  # Optional.
      control:                 # Optional.
      analytics_database:      # Optional.
      analytics:               # Optional.
      webui:                   # Optional.
      k8s_master:              # Optional.
      kubemanager:             # Optional.
   ip: BMS1_IP
  bms2:
   provider: bms
   roles:            # Optional.  If roles is not defined, all below roles will be created
     vrouter:        # Optional.
     k8s_node:       # Optional.
   ip: BMS2_IP
contrail_configuration:
   CONTRAIL_VERSION: latest
global_configuration:
   CONTAINER_REGISTRY: ci-repo.englab.juniper.net:5010
   REGISTRY_PRIVATE_INSECURE: True
```

### Step 5.	Turn off the swap functionality on all the nodes.
```
swapoff -a
```

### Step 6.	Configure the nodes.
```
ansible-playbook -e orchestrator=kubernetes -i inventory/ playbooks/configure_instances.yml
```
### Step 7.	Install Kubernetes and Contrail.
```
ansible-playbook -e orchestrator=kubernetes -i inventory/ playbooks/install_k8s.yml
ansible-playbook -e orchestrator=kubernetes -i inventory/ playbooks/install_contrail.yml
```
