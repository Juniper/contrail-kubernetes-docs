# Contrail + ( Kubernetes / Openshift )

Kubernetes (K8s) is an open source container management platform. It provides a portable platform across public and private clouds. K8s supports deployment, scaling and auto-healing of applications. More details can be found at: http://kubernetes.io/docs/whatisk8s/. 

There is a need to provide pod addressing, network isolation, policy based security, gateway, SNAT, loadalancer and service chaining capability in Kubernetes orchestratation. To this end K8s supports a framework for most of the basic network connectivity. This pluggable framework is called Container Network Interface (CNI). Opencontrail will support CNI for Kubernetes.

Currently K8s provides a flat networking model wherein all pods can talk to each other. Opencontrail will add additional networking functionality to the solution - multi-tenancy, network isolation, micro-segmentation with network policies, load-balancing etc. 

![Contrail Solution](images/standalone-kubernetes.png)

# Installation: Quick and Simple

***Disclaimer: 
The one-step install is for those who are waiting with bated breath to get their hands on a Contrail
Kubernetes cluster. This is meant as a quickstart install and is not as fequently validated to work
as the [standard mode](https://github.com/Juniper/contrail-kubernetes-docs/blob/master/README.md#installation-standard-and-recommended) of install. Though functionally identical to standard install, we dont recommend
this for anything other than quick tryouts.***

### [1-Step Standalone Kubernetes - Ubuntu](install/kubernetes/standalone-kubernetes-ubuntu.md)
### [1-Step Standalone Kubernetes - Centos](/install/kubernetes/standalone-kubernetes-centos.md)

# Installation: Standard and Recommended

## Deployment Modes - Kubernetes

Contrail provides more than one way of providing networking to a K8s cluster.

### [Standalone Kubernetes](install/kubernetes/standalone-kubernete-ansible.md)
### [Nested Kubernetes](install/kubernetes/nested-kubernetes.md)
### [Non-Nested Kubernetes](install/kubernetes/non-nested-kubernetes.md)

## Deployment Modes - Openshift

### [Standalone Openshift 3.11](install/openshift/3.11/standalone-openshift.md)
### [Nested Openshift 3.11](install/openshift/3.9/nested-mode-openshift-3.11.md)
### [Standalone Openshift 3.9](install/openshift/3.9/standalone-openshift.md)
### [Nested Openshift 3.9](install/openshift/3.9/nested-mode-openshift.md)
### [Standalone Openshift 3.7](install/openshift/3.7/standalone-openshift.md)
