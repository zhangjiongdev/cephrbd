
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
curl -o get-pip.py https://bootstrap.pypa.io/get-pip.py
python get-pip.py
sudo yum install yum-plugin-priorities -y
sudo yum install ceph-deploy -y
```
生成密钥对
```
ssh-keygen -t rsa

```
创建临时工作目录
```
mkdir my-cluster
cd my-cluster

```

# 配置 ceph1/ceph2/ceph3

```
ceph-deploy new ceph1

ceph-deploy install ceph1 ceph2 ceph3
```
```
#vi ceph.conf
public network = 30.0.0.0/24
mon clock drift allowed = 2
mon clock drift warn backoff = 30
```
```
ceph-deploy --overwrite-conf config push ceph1 ceph2 ceph3
```
```
ceph-deploy mon create-initial
ceph-deploy mon add ceph2
ceph-deploy mon add ceph3

ceph-deploy admin ceph1 ceph2 ceph3


ceph-deploy mgr create ceph1
ceph-deploy mgr create ceph2
ceph-deploy mgr create ceph3


ceph-deploy osd create --data /dev/sdb ceph1
ceph-deploy osd create --data /dev/sdb ceph2
ceph-deploy osd create --data /dev/sdb ceph3
```


ceph osd pool create rbd 128
rbd pool init rbd