1. Upload an image to Openstack.
   
    On the Openstack master node:
    
    a. Download the image 
    ```
       wget http://10.87.129.3/pxe/Standard/dpdkvm/vin/x-traffic1.qcow2
    ```
    b. Source openstack admin credentials
    ```
        source /etc/kolla/kolla-toolbox/admin-openrc.sh
    ```
    c. Upload the image using glance
    ```
       glance image-create --name contrail-u16.04 --visibility=public --disk-format qcow2  --container-format bare --file x-traffic1.qcow2
    ```

2. Create a compute flavor to use on Openstack GUI.

    a. On the Openstack UI:  ADMIN -> System -> Flavors
    
    b. Create Flavor
    
       Sample large flavor: VCPU: 4, RAM: 4096 MB, Root Disk: 75 GB
    
3. Create a Host Aggregate and Availability Zone to use on Openstack GUI.

    a. On the Openstack UI:  ADMIN -> System -> Host Aggregates
    
    b. Create Host Aggregate. 

        Remember to add compute Hosts to the Host Aggregate.
    
4. Import Key Pairs if need be. (Will be required for many scenarios)
    
    a. On the Openstack UI: Project -> Compute -> Key Pairs
    
    b. Import Key Pair.
    
       There are multiple ways to do this.

       . Import your mac's keypair, so you can access the VM directly. This works only if your VM can be reached directly from your mac.

       . If your VM does not have external connectivity and you usually ssh from computes using meta IP,
           one workflow could be to create to generate ssh-keypairs (i.e ssh-keygen) on all your computes and import
           their public keys together as a single key pair in Openstack.

5. Create a Virtual Network of your choice in Contrail GUI.

    a. On Contrail UI: Configure -> Networking -> Networks
    
    b. Create a Network.
    
    c. Enable SNAT on the Network
    
       i.  Network -> Edit -> Advanced Options -> Select SNAT
       ii. Save

6. Create VM instances.

    a. On the Openstack UI: Project -> Compute -> instances -> Launch Instance
    
    b. Fill out the following:
    ```
        . name
        . Availability Zone (created earlier)
        . Number of desired instances.
        . Image
        . Flavor
        . Network to use.
        . Key pair to use (created earlier)
    ```

7. Login to the Virtual Machines.

    a. Determine the compute host on which the VM is running, by going here.

       Project -> Compute -> Instances -> < Instance Name >
      
    b. Figure out the meta IP address for the instance.

       i.   On the Contrail GUI goto: Monitor -> Infrastructure -> Virtual Routers -> Host name
       ii.  In the instance column, look for the name of the Virtual Machine.
       iii. Expand the instance entry to figure out the meta_ip_addr.
       iv.  From the compute host, ssh to the meta_ip_addr

8. Create a K8s on the Virtual Machines.

    There are couple of options here:
   
    a. [1-step install](https://github.com/Juniper/contrail-kubernetes-docs/blob/master/install/kubernetes/standalone-kubernetes-ubuntu.md)
    
    b. [Ansible playbook](https://github.com/Juniper/contrail-kubernetes-docs/blob/master/install/kubernetes/standalone-kubernetes-ansible.md)
    
        In this case you need to run asible from a node or another virtual machine that can reach the k8s virtual machines.
       

9. [Install Nested Contrail cluster](https://github.com/Juniper/contrail-kubernetes-docs/blob/master/install/kubernetes/nested-kubernetes.md)



       
    
