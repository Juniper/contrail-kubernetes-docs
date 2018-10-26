
### Following steps are how you need to bring up your nodes before running ansible

* Reimage all your servers with : 

```shell
/cs-shared/server-manager/client/server-manager reimage --server_id server1 centos-7.5
```

* Setup environment(all nodes):
  
```shell
yum install vim git wget -y && wget -O /tmp/epel-release-latest-7.noarch.rpm https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm && rpm -ivh /tmp/epel-release-latest-7.noarch.rpm && yum update -y && yum install python-pip -y && pip install ansible==2.5.2 && yum install python-netaddr -y

yum install -y centos-release-openshift-origin
```
