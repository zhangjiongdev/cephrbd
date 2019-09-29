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
停止并禁用防火墙
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

yum 加速
```
curl -o /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

```


```
cat << EOM > /etc/yum.repos.d/ceph.repo
[ceph]
name=Ceph packages for x86_64
baseurl=http://mirrors.aliyun.com/ceph/rpm-luminous/el7/x86_64/
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://mirrors.aliyun.com/ceph/keys/release.asc
priority=1

[ceph-noarch]
name=Ceph noarch packages
baseurl=http://mirrors.aliyun.com/ceph/rpm-luminous/el7/noarch
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://mirrors.aliyun.com/ceph/keys/release.asc
priority=1

[ceph-source]
name=Ceph source packages
baseurl=http://mirrors.aliyun.com/ceph/rpm-luminous/el7/SRPMS
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://mirrors.aliyun.com/ceph/keys/release.asc
priority=1
EOM

```

```
yum clean all
yum makecache fast

```


时间同步
```
systemctl enable chronyd && systemctl start chronyd
chronyc sources

```
```
mkdir /root/.ssh
chmod 700 /root/.ssh
scp root@deploy:/root/.ssh/id_rsa.pub /root/.ssh/authorized_keys
```



配置cephdeploy
```
进入cephdeploy的配置步骤
```

```
ceph osd pool create rbd 128
rbd pool init rbd
```


```
// systemctl restart ceph-mon.target
```