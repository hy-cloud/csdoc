***安装前先规划好，如主机名，网络规划，RAID组成等***
# 1、RAID和BIOS界面的设置
>1. RAID
	+ 

>2. BIOS
	+ 开启硬件虚拟化


# 2、管理节点的安装
### 2.1、centos6的安装


### 2.2、cloudstack的安装
>1. 进入centos系统，打开终端
>2. vi /etc/sysconfig/network-scripts/ifcfg-eth0(为网卡配置已经规划好的IP地址，)
>3. service network restart
>4. vi /etc/hosts (修改主机名)
>5. vi /etc/sysconfig/network (修改HOSTNAME为“主机名+域名”，如： HOSTNAME=mgr1.cloud.com)
>6. 重启，检查主机名，输入hostname --fqdn （显示是否正确）
>7. vi /etc/selinux/config 改成SELINUX=permissive（修改selinux）
>8. service iptables stop  (关闭防火墙)
>9. 上传安装包至服务器/usr目录下，并解压配置本地yum源（记得授权）
>10. 


 
# 3、计算节点的安装
### 3.1、Xenserver
#### 3.1.1、xenserver的安装（需要开启硬件虚拟化）


### 3.2、网卡绑定及配置IP
>1. xe network-list
>2. xe network-param-set uuid=XXX name-label=cloud-all（XXX为管理网口的IP地址）
>3. xe network-create name-label=storage
>4. xe pif-list
>5. xe bond-create mode=lacp network-uuid=(storage网络的uuid) pif-uuids=(需要绑定的网卡的pif-uuid,用“，”隔开)
>6. xe host-list (找到主机的host-uuid)
>7. xe pif-list host-uuid=XXXX 
>8. xe pif-reconfigure-ip DNS=172.16.0.1 gateway=172.16.0.254 IP=172.16.0.11 mode=static netmask=255.255.255.0 uuid=XXXXX (XXXXX为上一步中找到的bond0的pif-uuid)



# 4、交换机的配置（以S5700为例）
### 4.1、添加用户及配置密码
>1. aaa    （进入aaa模式）
>2. local-user xuanyuan password simple 123456  （创建用户xuanyuan密码为123456，不加密）
>3. local-user xuanyuan password cipher 123456  （创建用户xuanyuan密码为123456，加密）
>4. local-user xuanyuan privilege level 15  (设置用户名为xuanyuan的管理级别为15)

### 4.2、vlan
>1. 连接并进入交换机（system-view）
>2. vlan 4000（创建vlan）
>3. port hybrid pvid vlan 4001（默认）
>4. port hybrid tagged vlan 100 to 1111
>5. port hybrid untagged vlan 10 40 4001

### 4.3、ssh远程Z
>1. stelnet server enable
>2. ssh user admin 
>3. ssh user admin authentication-type all
>4. ssh user admin server-type all 

### 4.4、snmp协议的开启
>1. 连接进入交换机（system-view）
>2. insterface METH0/0/1
>3. ip address 10.100.100.253 255.255.255.0
>4. snmp-agent sys-info version all
>5. snmp-agent community read cipher 123@Snmp v2c
>6. snmp-agent target-host trap address udp-admain 10.100.100.253 udp-port 161 params securityname cipher 123@Snmp v2c
>7. snmp-agent trap-enable

### 4.5、telnet配置
>1. interface vlanif 1
>2. ip address 10.100.100.252 255.255.255.0 （配置IP）
>3. user-interface vty 0 4 （设置用户从VTY 0-4用户界面登录）   
>4. shell（开启终端服务功能）
>5. protocol inbound telnet （配置用户界面支持Telnet服务）
>6. authentication-mode password （配置采用密码方式，aaa是账号加密码验证，none是不验证）
>7. set authentication password cipher xuanyuan (设置密码为xuanyuan)

### 4.6、路由添加
>1. ip route-stacic 0.0.0.0 0.0.0.0 192.168.29.254
>2. ip route 10.100.100.0 255.255.255.0 10.100.100.1


# 5、存储的配置
### 5.1、nfs
>1. 首先确定安装和启动nfs服务

### 5.2、ISCSI
>1. 存储服务器
	+ 

>2. 存储通道
	+ 


### 5.3、多路径




