
操作系统版本
```
cat /etc/redhat-release
```
OS: CentOS Linux release 7.7.1908 (Core)
内存: 512MB
CPU: 2

修改ip
```
vi /etc/sysconfig/network-scripts/ifcfg-enp0s3
```

修改主机名
```
hostnamectl set-hostname <hostname>
```
```
cat >> /etc/hosts << EOF
30.0.0.50 deploy
30.0.0.51 ceph1
30.0.0.52 ceph2
30.0.0.53 ceph3
EOF

```
停防火墙
```
systemctl stop firewalld && systemctl disable firewalld

```
禁用UseDNS
```
sed -i "s/#UseDNS yes/UseDNS no/g" /etc/ssh/sshd_config

```
禁用SELINUX
```
setenforce 0
sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config
reboot

```

