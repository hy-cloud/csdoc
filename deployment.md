#整个过程概述
该安装手册建设在CentOS6.5使用Xenserver一个云的cloudstack，做这一切在两个两个联想的RD640服务器（一台管理节点CentOS6.5系统和一台计算节点Xenserver6.2.0版本）上实现。Xenserver是一种虚拟化技术，主机的CPU处理器需支持硬件虚拟化扩展。

###先决条件
```
完成这个操作手册需要以下条件：
1、至少一台服务器（XenServer节点）支持开启硬件虚拟化
2、CentOS 6.5跟XenServer6.2.0的64位的安装光盘
3、网络规划
```

# 1、管理节点的安装
```
本次的管理节点使用Centos6.4的64的系统，为图形桌面安装。
IP：192.168.21.10
掩码：255.255.255.0
网关：192.168.21.254
```
### 1.1、centos6的安装

### 1.2、cloudstack的安装

##### 1.2.1、网络配置
使用root用户登录centos系统的本地控制台，检查并修改配置文件/etc/sysconfig/network-scripts/ifcfg-eth0，内容如下所示：
```
# vi /etc/sysconfig/network-scripts/ifcfg-eth0
```
```
DEVICE=eth0
HWADDR=52:54:00:B9:A6:C0
NM_CONTPROLLED=no
ONBOOT=yes
BOOTPROTO=none
IPADDR=192.168.21.240
NETMASK=255.255.255.0
GATEWAY=192.168.21.254
DNS1=202.96.128.86
```
配置文件修改完成后，需要运行命令重新启动网络。
```
# service network restart
# chkconfig network on
```
##### 1.2.2、主机名
cloudstack需要正确设置主机名，如果安装时默认，则主机名为：localhost.localdomain，需正确设置，编辑/etc/hosts文件在最后添加一行：
```
192.168.21.240 mgr1.cloud.com
```
编辑/etc/sysconfig/network文件，将内容更改为：
```
NETWORKING=yes
HOSTNAME=mgr1.cloud.com
```
需重启生效，重启之后使用如下命令进行验证：
```
# hostname --fqdn
```
返回结果为：
```
mgr1.coud.com
```
##### 1.2.3、SELINUX
当前的CloudStack需要将SELinux设置为permissive才能正常工作，您需要改变当前配置，同时将该配置持久化，使其在主机重启后仍然生效。在系统运行状态下将SELinux配置为permissive需执行如下命令：
```
# setenforce 0
```
为确保其持久生效需更改配置文件/etc/selinux/config，设置为permissive，如下例所示：
```
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
# enforcing - SELinux security policy is enforced.
# permissive - SELinux prints warnings instead of enforcing.
# disabled - No SELinux policy is loaded.
SELINUX=permissive
# SELINUXTYPE= can take one of these two values:
# targeted - Targeted processes are protected,
# mls - Multi Level Security protection.
SELINUXTYPE=targeted
```
##### 1.2.4、配置cloudstack软件库
添加CloudStack软件仓库，创建/etc/yum.repos.d/cloudstack.repo文件，并添加如下信息。
```
[cloudstack]
name=cloudstack
baseurl=http://cloudstack.apt-get.eu/centos/6/4.6/
enabled=1
gpgcheck=0
```
#####  1.2.5、nfs
本文档是配置使用NFS作为主存储和辅助存储，需配置两个NFS共享目录，在此之前需先安装nfs-utils：
```
# yum -y install nfs-utils
# mkdir /primary /secondary
```
接下来需配置NFS提供两个不同的挂载点。通过编辑/etc/exports文件即可简单实现。确保这个文件中包含下面内容：
```
/secondary *(rw,async,no_root_squash,no_subtree_check)
/primary *(rw,async,no_root_squash,no_subtree_check)
```

 
# 2、计算节点的安装
```
硬件必须是支持并开启虚拟化
本次的管理节点为XenServer6.2.0的64位系统，使用光盘安装
管理IP：192.168.21.11
掩码：255.255.255.0
网关：192.168.21.254
```
### 2.1、Xenserve
### 2.1.1、xenserver的安装（需要开启硬件虚拟化）


# 3、XenServer网卡绑定及配置IP
首先创建一个storage网络：
```
# xe network-create name-label=storage
```
下面的network-uuid为storage网络的uuid，pif-uuids后面接要绑定的网卡的uuid用‘，’隔开，使用xe pif-list命令可以查看：
```
# xe bond-create mode=lacp network-uuid=XXX pif-uuids=XXX
```

找到主机的pif-uuid：
```
# xe host-list
# xe pif-list host-uuid=XXXX 
```
给绑定网络设置静态IP：
```
xe pif-reconfigure-ip DNS=172.16.0.1 gateway=172.16.0.254 IP=172.16.0.11 mode=static netmask=255.255.255.0 uuid=XXXXX 
```


# 4、交换机的配置
```
以下交换机的配置都是使用华为S5700交换机
```
### 4.1、添加用户及配置密码
进入aaa模式
```
[S5700]aaa
```
创建用户xuanyuan密码为123456，不加密
```
[S5700-aaa]local-user xuanyuan password simple 123456
```
创建用户xuanyuan使其密码为123456，加密
```
[S5700-aaa]local-user xuanyuan password cipher 123456
```
设置用户名为xuanyuan的管理级别为15
```
[S5700-aaa]local-user xuanyuan privilege level 15
```


### 4.2、vlan
创建vlan：
```
[S5700]vlan 4000
```
给端口设置默认vlan以及标签：
```
port hybrid pvid vlan 4001
port hybrid tagged vlan 100 to 1111
port hybrid untagged vlan 10 40 4001
```
### 4.3、配置交换机ssh远程连接

```
stelnet server enable
ssh user admin 
ssh user admin authentication-type all
ssh user admin server-type all 
```
### 4.4、snmp协议的开启
进入交换机端口
```
[S5700]interface METH0/0/1
```
为交换机配置IP地址：
```
[S5700-METH0/0/1]ip address 10.100.100.253 255.255.255.0
```
snmp协议的开启以及连接设置：
```
[S5700]snmp-agent sys-info version all
[S5700]snmp-agent community read cipher 123@Snmp v2c
[S5700]snmp-agent target-host trap address udp-admain 10.100.100.253 udp-port 161 params securityname cipher 123@Snmp v2c
[S5700]snmp-agent trap-enable
```

### 4.5、telnet配置
Telnet连接服务，IP为10.100.100.252，密码为password：
```
[S5700]interface vlanif 1
[S5700]ip address 10.100.100.252 255.255.255.0 
[S5700]user-interface vty 0 4    
[S5700]shell
[S5700]protocol inbound telnet 
[S5700]set authentication password cipher password 
```
### 4.6、路由添加
默认路由：
```
ip route-stacic 0.0.0.0 0.0.0.0 192.168.29.254
```
一般路由：
```
ip route 10.100.100.0 255.255.255.0 10.100.100.1
```

# 5、存储的配置
### 5.2、ISCSI
1. 存储服务器	

2. 存储通道



### 5.3、多路径

# 1、RAID和BIOS界面配置
这里我是用联想服务器RD650为例：
1.1. RAID
	 

1.2. BIOS	
	