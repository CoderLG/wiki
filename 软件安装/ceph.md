
# 一、安装ceph集群

***该文档python版本为2.7，其他版本将造成osd一直为down***

***使用ceph-deploy安装，所有节点主机名与host要一致***

***
>**所有ceph节点**
***

## 0.修改hostname

```Shell
vi /etc/hosts
o
node1
node2
node3

/etc/init.d/network restart

hostnamectl set-hostname node3
```



## 1. 配置防火墙或者关闭

```Shell
#关闭
systemctl stop firewalld && systemctl disable firewalld
----或者----配置开放端口
#firewall-cmd --zone=public --add-port=6789/tcp #临时添加可用
firewall-cmd --zone=public --add-port=6789/tcp --permanent #永久，单必须临时添加可用
firewall-cmd --zone=public --add-port=123/udp #时间服务器需开放时间同步端口
firewall-cmd --zone=public --add-port=123/udp --permanent #时间服务器需开放时间同步端口
firewall-cmd --zone=public --add-port=6800-7300/tcp --permanent
firewall-cmd --reload # 重新加载 面向permanent
firewall-cmd --zone=public --list-all
                            查看
                            firewall-cmd --zone= public --query-port=80/tcp
                            删除
                            firewall-cmd --zone= public --remove-port=80/tcp --permanent

#检查----若是running（建议手动kill掉防火墙进程）
firewall-cmd --state
```

## 2. 关闭selinux

可能修改失败,可以vi进行修改

```Shell
sed -i "/^SELINUX/s/enforcing/disabled/" /etc/selinux/config
setenforce 0
```

## 3. 配置ntp时间同步服务

***不同步时间，ceph组件可能启动失败***

```Shell
#将时区修改成一样的
timedatectl
timedatectl set-timezone Asia/Shanghai

# 安装ntp
yum install ntp ntpdate

# 集群同时同步 网络时间
# ntpdate -u time.pool.aliyun.com

# 或者 以一台为时间服务器 其他服务器同步这个时间
# 停用其他时间服务器
sed -i '/^server/s/^/#/g' /etc/ntp.conf
# 同步自身时间
sed -i '25aserver 127.127.1.0\nfudge 127.127.1.0 stratum 8' /etc/ntp.conf

# 网络通的话 可以先同步一下网络时间 
# ntpdate -u time.pool.aliyun.com

# 启动
systemctl start ntpd
systemctl enable ntpd

#其他服务器同步上面ntp的服务时间
/usr/sbin/ntpdate 时间服务ip

# 机器启动时更新
echo "/usr/sbin/ntpdate 时间服务ip" >>/etc/rc.local
chmod +x /etc/rc.local

# 检查，*开头代表当前同步的时间服务器的ip
ntpq -pn

```

## 4. 添加 Ceph 源 (N版本-14)

***可进入[阿里源]( http://mirrors.aliyun.com/ceph/ )和 [163源](http://mirrors.163.com/ceph/ )选择其他版本***

```Shell
cat <<EOF > /etc/yum.repos.d/ceph.repo
[ceph]
name=ceph
baseurl=http://mirrors.aliyun.com/ceph/rpm-nautilus/el7/x86_64/
gpgcheck=0
priority=1

[ceph-noarch]
name=cephnoarch
baseurl=http://mirrors.aliyun.com/ceph/rpm-nautilus/el7/noarch/
gpgcheck=0
priority=1

[ceph-source]
name=Ceph source packages
baseurl=http://mirrors.163.com/ceph/rpm-nautilus/el7/SRPMS/
enabled=0
gpgcheck=1
type=rpm-md
gpgkey=http://mirrors.163.com/ceph/keys/release.asc
priority=1
EOF
```

## 5. 设置阿里云的base源和epel源

```Shell
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

Centos-7.repo文件内容:

```Shell
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the
# remarked out baseurl= line instead.
#
#

[base]
name=CentOS-$releasever - Base - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/os/$basearch/
    http://mirrors.aliyuncs.com/centos/$releasever/os/$basearch/
    http://mirrors.cloud.aliyuncs.com/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

#released updates
[updates]
name=CentOS-$releasever - Updates - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/updates/$basearch/
    http://mirrors.aliyuncs.com/centos/$releasever/updates/$basearch/
    http://mirrors.cloud.aliyuncs.com/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/extras/$basearch/
    http://mirrors.aliyuncs.com/centos/$releasever/extras/$basearch/
    http://mirrors.cloud.aliyuncs.com/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/centosplus/$basearch/
    http://mirrors.aliyuncs.com/centos/$releasever/centosplus/$basearch/
    http://mirrors.cloud.aliyuncs.com/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

#contrib - packages by Centos Users
[contrib]
name=CentOS-$releasever - Contrib - mirrors.aliyun.com
failovermethod=priority
baseurl=http://mirrors.aliyun.com/centos/$releasever/contrib/$basearch/
    http://mirrors.aliyuncs.com/centos/$releasever/contrib/$basearch/
    http://mirrors.cloud.aliyuncs.com/centos/$releasever/contrib/$basearch/
gpgcheck=1
enabled=0
gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
```

```Shell
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
```

epel-7.repo文件内容:

```Shell
[epel]
name=Extra Packages for Enterprise Linux 7 - $basearch
baseurl=http://mirrors.aliyun.com/epel/7/$basearch
failovermethod=priority
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7

[epel-debuginfo]
name=Extra Packages for Enterprise Linux 7 - $basearch - Debug
baseurl=http://mirrors.aliyun.com/epel/7/$basearch/debug
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=0

[epel-source]
name=Extra Packages for Enterprise Linux 7 - $basearch - Source
baseurl=http://mirrors.aliyun.com/epel/7/SRPMS
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=0
```



## 6.安装部署工具

（在root权限下执行如下操作）

```shell
yum makecache
yum install ceph-deploy

python -V
pip -V
yum install python-pip

ceph-deploy --version #（如果提示模块不存在，则是python<2.7>版本不匹配,或者pip<2.7>安装setuptools)
```



## 7. 添加用户

```Shell
useradd ceph
echo 'ceph' | passwd --stdin ceph
echo "ceph ALL = (root) NOPASSWD:ALL" > /etc/sudoers.d/ceph
chmod 0440 /etc/sudoers.d/ceph
# 配置sshd可以使用password登录----（可能修改失败）----可以vi进行修改（no改成yes）
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
systemctl reload sshd
# 配置sudo不需要tty----（可能修改失败）----可以vi进行修改（注释Default requiretty）
sed -i 's/Default requiretty/#Default requiretty/' /etc/sudoers
```

***
>**ceph管理节点**
***

## 8.  ssh免密

```Shell
su - ceph
ssh-keygen  #（默认一直回车，生成秘钥，已经生成可跳过）
ssh-copy-id ceph@node21
ssh-copy-id ceph@node22
ssh-copy-id ceph@node29


问题：
Warning: the ECDSA host key for 'node3' differs from the key for the IP address '192.168.4.78'

解决：
ssh-keygen -R 192.168.4.78

```

## 9. 创建集群

***不要使用sudo也不要使用root用户运行如下的命令***

```Shell
#su - ceph
mkdir my-cluster && cd my-cluster
```

## 10.创建3个monitor（添加使用create） 

***注意：新版本（L版本以后)需要先执行12步骤***

```Shell
ceph-deploy new node21 node22 node29
# 查看配置文件
ls -l
```

## 11.配置ceph.conf

```Shell
vi ./ceph.conf
    [global]
    ...
    # 如果有多个网卡，应该配置如下选项，
    # public network是公共网络，负责集群对外提供服务的流量
    # cluster network是集群网络，负载集群中数据复制传输通信等
    # 本次实验使用同一块网卡，生境环境建议分别使用一块网卡
    public network = 192.168.4.0/24
    cluster network = 192.168.4.0/24
```

***
>**所有ceph节点**
***

## 12. 安装 ceph 包

```Shell
# 如果按照官方文档安装方法 会重新配置安装官方ceph源
# 由于网络问题，安装可能会出错，需要多次执行
# ceph-deploy install 其实只是会安装 ceph ceph-radosgw 两个包
# ceph-deploy install node21 node22 node29
# 推荐使用阿里源安装，因为使用ceph-deploy安装会很慢
# 使用如下命令手动安装包，替代官方的 ceph-deploy install 命令
# 如下操作在所有node节点上执行
yum install ceph-release ceph ceph-radosgw
```

***
>**ceph管理节点**
***

## 13. 部署monitor和生成keys

```Shell
ceph-deploy mon create-initial
ls -l *.keyring
```

## 14. 复制文件到node节点

```Shell
ceph-deploy admin node1 node2 node3
```

## 15.添加osd

```Shell
# lsblk 列出所有磁盘
# 删除磁盘下分区 fdisk /dev/sdu
#          操作   d w
#
# 创建过osd后进入lvm卷删除--(也可用vgs+vgremove)
#    lsblk /dev/sdu
#    dmsetup remove ***
#
# pvdisplay查看vg卷对应
#
#####磁盘必须未挂载
for disk in /dev/sdu /dev/**
do
ceph-deploy disk zap node21 $disk
ceph-deploy osd create node21 --data $disk
done

for disk in /dev/sdd /dev/**
do
ceph-deploy disk zap node22 $disk
ceph-deploy osd create node22 --data $disk
done

for disk in /dev/sdn /dev/**
do
ceph-deploy disk zap node29 $disk
ceph-deploy osd create node29 --data $disk
done

```

### 单个节点添加自身的osd

```Shell
# 准备格式化LVM设备并将其与OSD关联:
ceph-volume lvm prepare --bluestore --data {device-path}
# 列出与Ceph相关的逻辑卷和设备;可用于查看{osd id} {osd fsid}
ceph-volume lvm list
# 激活发现并安装与OSD ID关联的LVM设备并启动Ceph OSD
ceph-volume lvm activate {osd-id}  {osd fsid}
# 当需要激活的osd较多时，可以一次性激活所有
ceph-volume lvm activate --all
```

>**可略**
>
>```Shell
># 在node节点创建目录
>## rm -rf /data/osd1
>## mkdir -pv /data/osd1
>## chmod 777 -R /data/osd1
>## chown ceph.ceph -R /data/osd1
># 添加osd 以文件目录方式
>## ceph-deploy osd prepare
>node21:/data/osd1 node22:/data/osd1 >node29:/data/osd1
>## ceph-deploy osd activate >node21:/data/osd1 node22:/data/osd1 >node29:/data/osd1
>```

## 16. 部署集群监控工具manager

>（luminous+）12及以后的版本必须部署才能使用

```Shell
# 自 nautilus 开始，dashboard作为一个单独的模块独立出来了，使用时需要单独安装yum install ceph-mgr-dashboard
ceph-deploy mgr create node1 node2 node3

# 开启dashboard模块，用于UI查看
## 管理KEY权限问题,普通用户无法读取导致无法进行cephx认证(但不推荐，这是集群管理秘钥）
## sudo chmod +r /etc/ceph/ceph.client.admin.keyring
# sudo ceph mgr module enable dashboard
# 监控网址
# curl http://192.168.4.21:7000

#启用
[root@controller ~]# ceph mgr module enable dashboard

#生成并安装一个 自签名证书
[root@node1 ~]# ceph dashboard create-self-signed-cert

Self-signed certificate created

# 生成密钥，生成两个文件——dashboard.crt、dashboard.key（可略）
[root@node1 ~]#mkdir mgr-dashboard
[root@node1 ~]#cd mgr-dashboard
[root@node1 ~]#openssl req -new -nodes -x509   -subj "/O=IT/CN=ceph-mgr-dashboard" -days 3650   -keyout dashboard.key -out dashboard.crt -extensions v3_ca

#配置服务地址（可以是0.0.0.0）、端口，默认的端口是8443/7000
[root@node1 mgr-dashboard]# ceph config set mgr mgr/dashboard/server_addr 0.0.0.0
[root@node1 mgr-dashboard]# ceph config set mgr mgr/dashboard/server_port 8443
[root@node1 mgr-dashboard]# ceph mgr services

{
    "dashboard": "https://0.0.0.0:8443/"
}

#创建一个用户、密码
[root@node1 mgr-dashboard]# ceph dashboard set-login-credentials admin admin

#重启
[root@node1 ~]# systemctl restart ceph-mgr@node1



```

## 17. 创建pool

```Shell
sudo ceph osd pool create rbd 128 128

# 激活存储池 （指定类型）
sudo ceph osd pool application enable rbd rgw

#如果不弄会出现问题
[root@k8s-product01-ceph01 /]# ceph health detail
HEALTH_WARN application not enabled on 1 pool(s)
POOL_APP_NOT_ENABLED application not enabled on 1 pool(s)
    application not enabled on pool 'sata-pool'
    use 'ceph osd pool application enable <pool-name> <app-name>', where <app-name> is 'cephfs', 'rbd', 'rgw', or freeform for custom applications.
# 这里的rbd是（块设备），pool 只能对一种类型进行 enable，另外两种类型是cephfs（文件系统），rgw（对象存储）

```

## 18. 查看状态

```Shell
## 管理KEY权限问题,普通用户无法读取导致无法进行cephx认证(但不推荐，这是集群管理秘钥）
## sudo chmod +r /etc/ceph/ceph.client.admin.keyring
sudo ceph health
sudo ceph -s
```

## 19. 清理集群

```Shell
# 如果安装过程出错，使用如下命令清理之后重新开始
ps aux|grep ceph | grep -v "grep"| awk '{print $2}'|xargs kill -9
ps -ef|grep ceph

配置文件存在：
ceph-deploy purge node1 node2 node3     #删除节点的所有的ceph数据包
ceph-deploy purgedata node1 node2 node3 #删除节点的所有的ceph数据包
ceph-deploy forgetkeys
rm ceph.*
若管理节点配置文件已经丢失：
sudo umount /var/lib/ceph/osd/*
sudo rm -rf /var/lib/ceph/osd/*
sudo rm -rf /var/lib/ceph/mon/*
sudo rm -rf /var/lib/ceph/mds/*
sudo rm -rf /var/lib/ceph/bootstrap-mds/*
sudo rm -rf /var/lib/ceph/bootstrap-osd/*
sudo rm -rf /var/lib/ceph/bootstrap-rgw/*
sudo rm -rf /var/lib/ceph/tmp/*
sudo rm -rf /etc/ceph/*
sudo rm -rf /var/run/ceph/*　　
```

# 二、安装客户端

## 1. 验证是否符合块设备环境要求

```Shell
 uname -r      #------3.10***
 modprobe rbd  #------
 echo $?       #------0
```

## 2.只安装ceph

```Shell
yum install ceph

# 拷贝集群的ceph.conf---不拷贝秘密信息
# 和创建秘钥
# ceph auth get-or-create client.rbd mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=rbd' |tee ./ceph.client.rbd.keyring
# 拷贝到 /etc/ceph/

ceph -s --name client.rbd
```

# 三、 配置块设备（RBD）

***
>**ceph节点**
***

## 1. 创建池和块

```Shell
ceph osd lspools  #查看集群存储池
ceph osd pool create rbd 512 #512为pg数量
```

## 2. pg_num取值需要计算，常用值

```Shell
#0<osd<5 128
#5<=osd<10 512
#10<=osd<50 4096
#50<osd #通过权衡方法计算
```

***
>**客户端/ceph节点**
***

## 3.创建块设备

```Shell
#--pool rbd --image rbd1 --image-format 2 --image-feature layering --size 10G
rbd create rbd1 --size 10240 -n client.rbd

rbd ls --name client.rbd
rbd ls -p rbd --name client.rbd
rbd list --name client.rbd

rbd --image rbd1 info -n client.rbd  #镜像
​```Shell

 ***
>**客户端**
***

## 4. 映射块设备
​```Shell
rbd map --image rbd1 --name client.rbd
    **layering:分层支持
    **exclusive-lock:排它锁支持对
    **object-map:对象映射支持（需要exclusive-lock）
    **deep-flatten:快照平支持（snapshot flatten support）
    **fast-diff:在client-node上使用krbd（内核rbd）客户机进行快速diff计算（需要object-map),无法在centos内核3.10上映射块设备镜像，
    **因为不支持object-map、deep-flatten和fast-diff(在内核4.9中引入）。
    **所以要进行禁用不支持的特性
    #动态禁用
    rbd feature disable rbd1 exclusive-lock object-map deep-flatten fast-diff --name client.rbd
    #创建时只启用 分层特性
    rbd create rbd1 --size 10240 --image-feature layering --name client.rbd
    #ceph配置文件中禁用
        rbd_default_features=1
rbd showmapped --name client.rbd
```

## 5. 创建文件系统 并挂载

```Shell
fdisk -l /dev/rbd0
mkfs.xfs /dev/rbd0
mkdir /mnt/ceph-disk1
mount /dev/rbd0 /mnt/ceph-disk1
df -h /mnt/ceph-disk1
```

## 6. 做成服务，开机自动挂载

```Shell
#已改进
wget -O /usr/local/bin/rbd-mount https://raw.githubusercontent.com/aishangwei/ceph-demo/master/client/rbd-mount
	#!/bin/bash

	#exit if any err throw
	set -o errexit

	# Pool name where block device image is stored
	export poolname=rbd

	# Keyring name
	export clientname="client.rbd"
	 
	# Disk image name
	export rbdimage=rbd1
	 
	# Mounted Directory
	export mountpoint=/mnt/ceph-disk1

	# can run rbd?
	modprobe rbd
	 
	# Image mount/unmount and pool are passed from the systemd service as arguments
	# Are we are mounting or unmounting

	# image is exist?
	export imageisexist=`rbd ls -n $clientname |grep $rbdimage |awk '{var=$1;print $var}'`
	[[ ! $imageisexist ]] && { echo "image ‘$rbdimage’ is not exist"; exit 1; } || {
		# mount
		if [ "$1" == "m" ]; then
		   set +e
		   rbd feature disable $rbdimage object-map fast-diff deep-flatten --name $clientname 2>/dev/null
		   set -e
		   # driver name
		   export driver=`rbd map --image $rbdimage --id rbd --name $clientname`
		   mkdir -p $mountpoint
		   [[ $driver ]] && mount $driver $mountpoint || rbd unmap $driver --name $clientname
		fi
		# umount
		if [ "$1" == "u" ]; then
		   # driver name
		   export driver=`df -h|grep $mountpoint|awk '{print $1}'|awk '{var=$1;print $var}'`
		   umount $mountpoint || rbd unmap $poolname/$rbdimage@-
		   [[ $driver ]] && rbd unmap $driver --name $clientname
		fi
		# remount
		if [ "$1" == "r" ]; then
		   # umount
		   # driver name
		   export driver=`df -h|grep $mountpoint|awk '{print $1}'|awk '{var=$1;print $var}'`
		   umount $mountpoint || rbd unmap $poolname/$rbdimage@-
		   [[ $driver ]] && rbd unmap $driver --name $clientname
		   # mount
		   set +e
		   rbd feature disable $rbdimage object-map fast-diff deep-flatten --name $clientname 2>/dev/null
		   set -e
		   # driver name
		   export driver=`rbd map --image $rbdimage --id rbd --name $clientname`
		   mkdir -p $mountpoint
		   [[ $driver ]] && mount $driver $mountpoint || rbd unmap $driver --name $clientname
		fi
	}

chmod +x /usr/local/bin/rbd-mount

wget -O /etc/systemd/system/rbd-mount.service  https://raw.githubusercontent.com/aishangwei/ceph-demo/master/client/rbd-mount.service
    [Unit]
    Description=RADOS block device mapping for $rbdimage in pool $poolname"
    Conflicts=shutdown.target
    Wants=network-online.target
    After=NetworkManager-wait-online.service
    [Service]
    Type=oneshot
    RemainAfterExit=yes
    TimeoutSec=0 # 单位是秒，默认是0不限制
    ExecStart=/usr/local/bin/rbd-mount m
    ExecStop=/usr/local/bin/rbd-mount u
    ExecReload=/usr/local/bin/rbd-mount r
    [Install]
    WantedBy=multi-user.target

systemctl daemon-reload
systemctl enable rbd-mount.service
df -h /mnt/ceph-disk1
```

# 四、配置对象存储（RGW）

RESTful s3/swift兼容api

多个RGW（可以配置负载均衡器）
***
>**ceph节点**
***

## 1.安装 

 该文档前面已经安装过。

 ```Shell
#yum install ceph-radosgw
 ```

## 2. 部署

```Shell
ceph-deploy rgw create node1 node2 node3
```

## 3. 配置80端口(默认7480）

>可略过该部分

```Shell
>vi /etc/ceph/ceph.conf
>...
>[client.rgw.node22]
>rgw_frontends="civetweb port=80"
>
>sudo systemctl restart ceph-radosgw@rgw.node22.service
```

## 4. 创建池

>可略

```Shell
wget https://raw.githubusercontent.com/aishangwei/ceph-demo/master/ceph-deploy/rgw/pool
```

pool文件内容：

```Shell
.rgw
.rgw.root
.rgw.control
.rgw.gc
.rgw.buckets
.rgw.buckets.index
.rgw.buckets.extra
.log
.intent-log
.usage
.users
.users.email
.users.swift
.users.uid
```

```Shell
wget https://raw.githubusercontent.com/aishangwei/ceph-demo/master/ceph-deploy/rgw/create_pool.sh
```

create_pool.sh文件内容：

```Shell
#!/bin/bash

PG_NUM=250
PGP_NUM=250
SIZE=3

for i in `cat pool`
    do
    ceph osd pool create $i $PG_NUM
    ceph osd pool set $i size $SIZE
    done

for i in `cat pool`
    do
    ceph osd pool set $i pgp_num $PGP_NUM
    done
```

```Shell
#执行
chmod +x create_pool.sh
./create_pool.sh
```

## 5. 测试

```Shell
ceph -s -k /var/lib/ceph/radosgw/ceph-rgw.node22/keyring --name client.rgw.node22
```

## 6. 使用s3 api访问RGW

### 6.1 创建radosgw用户

***
>**ceph节点**
***

```Shell
radosgw-admin user create --uid=radosgw --display-name="radosgw"
注意：把access_key和secret_key保存下来，忘记时使用：radosgw-admin user info --uid radosgw (-k /var/lib/ceph/radosgw/ceph-rgw.node22/keyring --name client.rgw.node22)
```

### 6.2 安装s3cmd

***
>**客户端**
***

```Shell
yum install s3cmd
# 将会在家目录下创建 .s3cfg文件 （Use HTTPS protocol：no)
s3cmd --configure

# 编辑.s3cfg文件，修改host_base和host_bucket
vi .s3cfg
    ...
    host_base=node21:7480
    host_bucket=%(bucket).node21:7480
    ...

# 创建桶 并加入
s3cmd mb s3://first-bucket

s3cmd ls
s3cmd put -r /etc/hosts s3://first-bucket

s3cmd ls s3://first-bucket

s3cmd get -r s3://first-bucket/  /home

s3cmd del s3://first-bucket/*
s3cmd rb s3://first-bucket
s3cmd ls
```

## 7. 使用swift api访问RGW

### 7.1 创建用户，默认启用

***
>**ceph节点**
***

```Shell
radosgw-admin subuser create --uid=radosgw --subuser=radosgw:swift --display-name="radosgw" --access=full #(--access=[ read | write | readwrite | full ])
注意：把access_key和secret_key保存下来，忘记时使用：radosgw-admin user info --uid radosgw (-k /var/lib/ceph/radosgw/ceph-rgw.node22/keyring --name client.rgw.node22)

# 停用用户
radosgw-admin user suspend --uid=radosgw
# 启用用户
radosgw-admin user enable --uid=radosgw
# 删除用户
radosgw-admin user rm --uid=radosgw
```

***
>**客户端**
***

### 7.2 安装seift

```Shell
    若缺少pip：(python2.7）
        yum install python-pip

        或者：

        wget https://bootstrap.pypa.io/get-pip.py  --no-check-certificate
        python2.7 ./get-pip.py
        ./pip2.7 install requests

pip install --upgrade python-swiftclient
#访问
swift -A http://node21:7480/auth/1.0 -U radosgw:swift -K ... list
swift -A http://node21:7480/auth/1.0 -U radosgw:swift -K ... post second-bucket
swift -A http://node21:7480/auth/1.0 -U radosgw:swift -K ... list
```

# 五、 部署cephfs与挂载

## 1 创建mds

***
>**ceph节点**
***

```Shell
ceph-deploy mds create node21

注意：查看输出，应该能看到执行了那些命令，以及keyring

ceph osd pool create cephfs_data 128
ceph osd pool create cephfs_metadata 64
ceph fs new cephfs cephfs_metadata cephfs_data

ceph mds stat
ceph osd pool ls
ceph fs ls
```

## 2 用户

（可选，部署时以生成）

```Shell
ceph auth get-or-create client.cephfs mon 'allow r' mds 'allow r,allow rw path=/' osd 'allow rw pool=cephfs_data' -o ceph.client.cephfs.keyring

scp ceph.client.cephfs.keyring 客户机host:/etc/ceph/
# 获取秘钥
ceph auth get-key client.cephfs
```

***
>**客户端**
***

## 3 内核挂载cephfs（2.6.34之后）

```Shell
# 创建挂载目录
mkdir /mnt/cephfs

# 挂载
mount -t ceph node21:6789:/ /mnt/cephfs -o name=cephfs,secret=...

echo ....secret...>/etc/ceph/cephfskey  #保存key
mount -t ceph node21:6789:/ /mnt/cephfs -o name=cephfs,secretfile=/etc/ceph/cephfskey    #name为用户

# 查看、卸载
df -h /mnt/cephfs
umount /mnt/cephfs

# 设置简洁挂载方式
echo "node21:6789:/ /mnt/cephfs ceph name=cephfs,secretfile=/etc/ceph/cephfskey,_netdev,noatime 0 0">> /etc/fstab

# 校验
umount /mnt/cephfs
df -h /mnt/cephfs
mount /mnt/cephfs

dd if=/dev/zero of=/mnt/cephfs/file1 bs=1M count=1024

```

## 4 fuse挂载cephfs

```Shell
# 安装软件包
#rpm -qa | grep -i ceph-fuse
yum install ceph-fuse

# 创建挂载目录
mkdir /mnt/cephfs-fuse

# 挂载
ceph-fuse --keyring /etc/ceph/ceph.client.cephfs.keyring --name client.cephfs -m node21:6789 /mnt/cephfs-fuse

# 设置简洁挂载方式
echo "id=cephfs,keyring=/etc/ceph/ceph.client.cephfs.kering /mnt/cephfs-fuse fuse.ceph defaults 0 0 _netdev">>/etc/fstab
  注：因为keyring文件包含了地址，所以fstab不需要指定地址了

# 校验
umount /mnt/cephfs-fuse
mount /mnt/cephfs-fuse
```

## 5. 导出ceph fs 为nfs服务器

### 5.1 安装软件 nfs

***
>**nfs服务节点**
***
[参考网址](https://blog.csdn.net/qq_38265137/article/details/83146421)

```Shell
yum install nfs-utils nfs-ganesha

# 启动nfs所需的rpc服务
systemctl start rpcbind
systemctl enable rpcbind
systemctl start rpc-statd.service

# 修改配置
vi /etc/ganesha/ganesha.conf

    EXPORT
    {
        ## Export Id (mandatory, each EXPORT must have a unique Export_Id)
        Export_Id=77;

        ## Exported path (mandatory)
        Path = "/";

        ## Pseudo Path (required for NFSv4 or if mount_path_pseudo = true)
        pseudo = "/";

        ## Access type for clients.  Default is None, so some access must be
        ## given. It can be here, in the EXPORT_DEFAULTS, or in a CLIENT block
        Access_Type = RW;

        ## Allowed security types for this export :sys,krb5,krb5i,krb5p;
        Sectype = "none";

        ## Restrict the protocols that may use this export.  This cannot allow
        ## access that is denied in NFS_CORE_PARAM :3,4;
        Protocols = "3";

        ## Whether to squash various users :root_squash;
        Squash = No_ROOT_Squash;

        ## Exporting FSAL
        FSAL{
            Name = CEPH
        }
    }
```

### 5.2 通过提供Ganesha.conf 启动NFS Ganesha守护进程

```Shell
ganesha.nfsd -f /etc/ganesha/ganesha.conf -L /var/log/ganesha.log -N NIV_DEBUG
showmount -e
```

***
>**客户端**
***

### 5.3 挂载nfs

```Shell
yum install nfs-utils
mkdir /mnt/ceph-nfs
mount -o rw,noatime node21:/ /mnt/ceph-nfs
```

# 六、其他操作

```Shell

ceph osd lspools
ceph osd pool set .rgw.root size 1
ceph osd pool set .rgw.root pgp_num 8
ceph osd pool set .rgw.root pg_num 8


ceph osd pool set default.rgw.control size 1
ceph osd pool set default.rgw.control pgp_num 8
ceph osd pool set default.rgw.control pg_num 8

ceph osd pool set default.rgw.meta size 1
ceph osd pool set default.rgw.meta pgp_num 8
ceph osd pool set default.rgw.meta pg_num 8

ceph osd pool set default.rgw.buckets.data size 1
ceph osd pool set default.rgw.buckets.data pgp_num 8
ceph osd pool set default.rgw.buckets.data pg_num 8


ceph osd pool set default.rgw.buckets.index size 1



#将配置文件推到mon所在的其他节点
ceph-deploy --overwrite-conf config push ceph1 ceph2

# 重启当前节点ceph-mon服务：
systemctl restart ceph-mon.target
# 重启当前节点中mon.node21的ceph-mon服务（如果mon.node21在当前节点）
systemctl restart ceph-osd@node21

# 重启当前节点ceph-mgr服务：
systemctl restart ceph-mgr.target
# 重启当前节点中mgr.node21的ceph-mgr服务（如果mgr.node21在当前节点）
systemctl restart ceph-mgr@node21

# 重启当前节点ceph-mgr服务：
systemctl restart ceph-mds.target
# 重启当前节点中mds.node21的ceph-mds服务（如果mds.node21在当前节点）
systemctl restart ceph-mds@node21

# 重启当前节点所有ceph-osd服务：
systemctl restart ceph-osd.target
# 重启当前节点中osd.0的ceph-osd服务（如果osd.0在当前节点）
systemctl restart ceph-osd@0

# 创建桶
ceph osd crush add-bucket node21-2 host
# 放到root下
ceph osd crush move node21-2 root=rbd
# osd加入crushmap
ceph osd crush add osd.4 0.00980 host=node21-2

# 创建秘钥
ceph auth get-or-create client.rbd mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=rbd' |tee ./ceph.client.rbd.keyring

# 为osd.0创建一个用户并创建一个key
ceph auth get-or-create osd.0 mon 'allow rwx' osd 'allow *' -o /var/lib/ceph/osd/ceph-0/keyring

# 为mds.node1创建一个用户并创建一个key
ceph auth get-or-create mds.node1 mon 'allow rwx' osd 'allow ' mds 'allow ' -o /var/lib/ceph/mds/ceph-node1/keyring

# 查看ceph集群中的认证用户及相关的key
ceph auth list

# 删除集群中的一个认证用户
ceph auth del osd.0

# 删除一个mon节点
ceph mon remove node21

# 查看最大osd的个数
ceph osd getmaxosd
    max_osd = 4 in epoch 514 #默认最大是4个osd节点

# down掉osd.0节点
ceph osd down 0

# 设置osd crush的权重为1.0
ceph osd crush set 3 3.0 host=node21

# 把一个osd逐出集群
ceph osd out osd.3
# 删除osd(ll /var/lib/ceph/osd/ceph-3/ 查看块（lvm）对应)
ceph osd rm osd.3
# 从crush map中移除osd
ceph osd crush remove osd.3
# 把逐出的osd加入集群
ceph osd in osd.3
# 暂停osd （暂停后整个集群不再接收数据）
ceph osd pause
# 再次开启osd （开启后再次接收数据）
ceph osd unpause

# 为存储池设置pg_num 和设置pgp_num默认 3
ceph osd pool set volumes pg_num 512
ceph osd pool set volumes pgp_num 512

# 设置存储池副本数
ceph osd pool get rbd size
ceph osd pool set rbd size 3

##创建 删除 存储池
# 创建pool
ceph osd pool create testPool 64

# 重命名pool
ceph osd pool rename testPool amizPool

# 获取pool 副本数 默认3
ceph osd pool get amizPool size
# 设置pool 副本数 默认3
ceph osd pool set amizPool size 3

# 获取pool pg_num/pgp_num
ceph osd pool get amizPool pg_num
ceph osd pool get amizPool pgp_num
# 设置pool pg_num/pgp_num
ceph osd pool set amizPool pg_num 128
ceph osd pool set amizPool pgp_num 128

# 删除存储池
ceph osd pool delete  amizPool  amizPool --yes-i-really-really-mean-it
# 打开mon节点的配置文件：
vi /etc/ceph/ceph.conf
    [mon]
    mon_allow_pool_delete = true
# 重启单前节点所有ceph-mon服务：
systemctl restart ceph-mon.target
# 删除池提示错误
    Error EBUSY: pool 'testpool' is in use by CephFS
# 从mds（cephfs)中移除存储池
ceph mds remove_data_pool testpool
ceph osd pool delete testpool testpool  --yes-i-really-really-mean-it

# 设置存储池配额
ceph osd pool set-quota rbd max_objects 10
    set-quota max_objects = 10 for pool rbd
ceph osd pool get-quota rbd
    quotas for pool 'rbd':
      max objects: 10 objects
      max bytes  : N/A

#取消配额限制
ceph osd pool set-quota rbd max_objects 0
    set-quota max_objects = 0 for pool rbd
ceph osd pool get-quota rbd
    quotas for pool 'rbd':
      max objects: N/A
      max bytes  : N/A

# 查看存储池pool 配额
ceph osd pool get-quota poolroom1
    quotas for pool 'poolroom1':
      max objects: N/A
      max bytes  : 6144MB   # 存储池pool配额 6G

# 激活存储池
ceph osd pool application enable .rgw.root rgw
    enabled application 'rgw' on pool '.rgw.root'


##编译map

#修改配置文件,防止ceph自动更新crushmap
#echo 'osd_crush_update_on_start = false' >> /etc/ceph/ceph.conf
#/etc/init.d/ceph restart

# 获取crush map
ceph osd getcrushmap -o /opt/getcrushmap/crushmap

# 反编译crush map
crushtool -d crushmap -o decrushmap

# 修改crush map
    在root default 后面添加下面两个bucket

    root ssd {
        id -5
        alg straw
        hash 0
        item osd.0 weight 0.01
    }
    root stat {
        id -6
        alg straw
        hash 0
        item osd.1 weight 0.01
    }
    在rules部分添加如下规则：

    rule ssd{
        ruleset 1
        type replicated
        min_size 1
        max_size 10
        step take ssd
        step chooseleaf firstn 0 type osd
        step emit
    }
    rule stat{
        ruleset 2
        type replicated
        min_size 1
        max_size 10
        step take stat
        step chooseleaf firstn 0 type osd
        step emit
    }

# 编译crush map
crushtool -c decrushmap -o newcrushmap

# 注入crush map
ceph osd setcrushmap -i /opt/getcrushmap/newcrushmap
    set crush map
    ceph osd tree
    ID WEIGHT  TYPE NAME           UP/DOWN REWEIGHT PRIMARY-AFFINITY
    -6 0.00999 root stat
     1 0.00999     osd.1                up  1.00000          1.00000
    -5 0.00999 root ssd
     0 0.00999     osd.0                up  1.00000          1.00000
    -1 0.58498 root default
    -2 0.19499     host ceph-admin
     2 0.19499         osd.2            up  1.00000          1.00000
    -3 0.19499     host ceph-node1
     0 0.19499         osd.0            up  1.00000          1.00000
    -4 0.19499     host ceph-node2
     1 0.19499         osd.1            up  1.00000          1.00000
    # 重新查看osd tree 的时候已经看见这个树已经变了。添加了名称为stat和SSD的两个bucket

# 创建资源池
ceph osd pool create ssd_pool 8 8
    pool 'ssd_pool' created
ceph osd pool create stat_pool 8 8
    pool 'stat_pool' created
ceph osd dump|grep ssd
    pool 28 'ssd_pool' replicated size 3 min_size 2 crush_ruleset 0 object_hash rjenkins pg_num 8 pgp_num 8 last_change 2484 flags hashpspool stripe_width 0
ceph osd dump|grep stat
    pool 29 'stat_pool' replicated size 3 min_size 2 crush_ruleset 0 object_hash rjenkins pg_num 8 pgp_num 8 last_change 2486 flags hashpspool stripe_width 0
注意：刚刚创建的两个资源池ssd_pool 和stat_pool 的 crush_ruleset  都是0，下面需要修改。

# 修改资源池存储规则
ceph osd pool set ssd_pool crush_ruleset 1
    set pool 28 crush_ruleset to 1
ceph osd pool set stat_pool crush_ruleset 2
    set pool 29 crush_ruleset to 2
ceph osd dump|grep ssd
    pool 28 'ssd_pool' replicated size 3 min_size 2 crush_ruleset 1 object_hash rjenkins pg_num 8 pgp_num 8 last_change 2488 flags hashpspool stripe_width 0
ceph osd dump|grep stat
    pool 29 'stat_pool' replicated size 3 min_size 2 crush_ruleset 2 object_hash rjenkins pg_num 8 pgp_num 8 last_change 2491 flags hashpspool stripe_width 0

# luminus 版本设置pool规则的语法是
ceph osd pool set ssd crush_rule ssd
    set pool 2 crush_rule to ssd
ceph osd pool set stat crush_rule stat
    set pool 1 crush_rule to stat

# 重新对pool授权(如果对接过openstack)

ceph auth del client.cinder
ceph auth get-or-create client.cinder mon 'allow r'
    osd 'allow class-read object_prefix rbd_children,allow rwx pool=volumes, allow rwx pool=vms, allow rx pool=images, allow rwx pool=ssd'

# 验证
    验证前先看看ssd_pool 和stat_pool 里面是否有对象
rados ls -p ssd_pool
rados ls -p stat_pool
# 这两个资源池中都没有对象
    用rados命令 添加对象到两个资源池中
    rados -p ssd_pool put test_object1 /etc/hosts
    rados -p stat_pool put test_object2 /etc/hosts
    rados ls -p ssd_pool
        test_object1
    rados ls -p stat_pool
        test_object2
    # 对象添加成功
ceph osd map ssd_pool test_object1
    osdmap e2493 pool 'ssd_pool' (28) object 'test_object1' -> pg 28.d5066e42 (28.2) -> up ([0], p0) acting ([0,1,2], p0)
ceph osd map stat_pool test_object2
    osdmap e2493 pool 'stat_pool' (29) object 'test_object2' -> pg 29.c5cfe5e9 (29.1) -> up ([1], p1) acting ([1,0,2], p
```

	ceph osd set noup      # prevent OSDs from getting marked up
	ceph osd set nodown    # prevent OSDs from getting marked down
# 禁止 ceph 自动迁移数据
	ceph osd set noout
	ceph osd set nobackfill
	ceph osd set norecover

# 此时能看到这三个 flag，而且集群处于不健康状态
ceph -s 

	ceph osd unset noup
	ceph osd unset nodown
# 取消之前设置的那三个 noflag
	ceph osd unset noout
	ceph osd unset nobackfill
	ceph osd unset norecover

# 在任一节点上执行下面命令，观察数据迁移
ceph -w