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
#openshift_repos_enable_testing=true
containerized=false
openshift_install_examples=true
openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider', 'filename': '/etc/origin/master/htpasswd'}]
osm_cluster_network_cidr=10.32.0.0/12
openshift_portal_net=10.96.0.0/12
openshift_use_dnsmasq=true
openshift_clock_enabled=true
openshift_hosted_manage_registry=true
openshift_hosted_manage_router=true
openshift_enable_service_catalog=false
openshift_use_openshift_sdn=false
os_sdn_network_plugin_name='cni'
openshift_disable_check=memory_availability,package_availability,disk_availability,package_version,docker_storage
openshift_docker_insecure_registries=ci-repo.englab.juniper.net:5000

openshift_use_contrail=true
contrail_version=5.0
contrail_container_tag=ocata-5.0-156
contrail_registry=ci-repo.englab.juniper.net:5000
# Username /Password for private Docker regiteries
#contrail_registry_username=test
#contrail_registry_password=test
# Provide this option if you want contrail to be installed on this interface on all nodes 
#vrouter_physical_interface=ens160
vrouter_gateway=10.84.13.254
#docker_version=1.13.1

[masters]
10.84.13.51 openshift_hostname=a6s41node1

[etcd]
10.84.13.51 openshift_hostname=a6s41node1

[nodes]
10.84.13.51 openshift_hostname=a6s41node1
10.84.13.52 openshift_hostname=a6s41node2

[openshift_ca]
10.84.13.51 openshift_hostname=a6s41node1

# Contrail installed on this subnet / vrouter_physical_interface takes precedence over this
[contrail_masters]
10.84.13.51 openshift_hostname=a6s41node1

[contrail_vars]
#kubernetes_api_server=20.1.1.1
#kubernetes_api_port=8080
#kubernetes_api_secure_port=8443
#cluster_name=k8s
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
