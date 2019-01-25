```subscription-manager register --username <> --password <> --force
subscription-manager attach --pool=<>

subscription-manager repos --disable="*"

subscription-manager repos \
    --enable="rhel-7-server-rpms" \
    --enable="rhel-7-server-extras-rpms" \
    --enable="rhel-7-server-ose-3.11-rpms" \
    --enable=rhel-7-fast-datapath-rpms \
    --enable="rhel-7-server-ansible-2.6-rpms"

yum install -y tcpdump wget git net-tools bind-utils yum-utils iptables-services bridge-utils bash-completion kexec-tools sos psacct python-netaddr
```
