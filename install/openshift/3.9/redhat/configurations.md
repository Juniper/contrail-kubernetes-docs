
### Following steps are how you need to bring up your nodes before running ansible

* Reimage all your servers with :

```shell
/cs-shared/server-manager/client/server-manager reimage --server_id server1 centos-7.5
```

* Setup environment(all nodes):
     * Register all nodes in cluster using Red Hat Subscription Manager (RHSM)
 ```shell
 (all-nodes)# subscription-manager register --username <username> --password <password> --force
 ```
   * List the available subscriptions
 ```shell
 (all-nodes)# subscription-manager list --available --matches '*OpenShift*'
 ```
   * From the previous command, find the pool ID for OpenShift Container Platform subscription & attach it
 ```shell
 (all-nodes)# subscription-manager attach --pool=<pool-ID>
 ```
   * Disable all yum respositories
 ```shell
 (all-nodes)# subscription-manager repos --disable="*"
 ```
   * Enable only the repositories required by OpenShift Container Platform 3.9
 ```shell
  subscription-manager repos \
    --enable="rhel-7-server-rpms" \
    --enable="rhel-7-server-extras-rpms" \
    --enable="rhel-7-server-ose-3.9-rpms" \
    --enable="rhel-7-fast-datapath-rpms" \
    --enable="rhel-7-server-ansible-2.5-rpms"
 ```
 ** Install EPEL
 ```shell
 (all-nodes)# yum install wget -y && wget -O /tmp/epel-release-latest-7.noarch.rpm https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm && rpm -ivh /tmp/epel-release-latest-7.noarch.rpm
 ```
 ** Update the system to use the latest packages
 ```shell
 (all-nodes)# yum update -y
 ```
** Install the following package, which provides OpenShift Container Platform utilities
 ```shell
 (all-nodes)# yum install atomic-openshift-excluder atomic-openshift-utils git python-netaddr -y
 ```
** Remove the atomic-openshift packages from the list for the duration of the installation
 ```shell
 (all-nodes)# atomic-openshift-excluder unexclude -y
 ```
** Enforce SELinux security policy
 ```shell
 (all-nodes)# vi /etc/selinux/config
        SELINUX=enforcing
 ```
