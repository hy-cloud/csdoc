##Clvm

### 1 Clvm 的架构

![pic]( https://raw.githubusercontent.com/hy-cloud/csdoc/master/pic/clvm.png )

### 2. 安装 Clvm

```
  yum install cman rgmanager lvm2-cluster
```
cman : 是一个集群管理工具
lvm2-cluster: lvm cluster


### 3. 配置 Clvm
vi /etc/lvm/lvm.conf
找到locking_type 修改 locking_type = 3
或者使用 lvmconf --enable-cluster 命令启用 集群lvm

使用 ccs_tool 添加集群节点, 在一个集群节点上执行如下命令：

```
ccs_tool create tcluster
ccs_tool create fence meatware fence_manual
ccs_tool addnode -n 1 -f meatware kvmnode1
ccs_tool addnode -n 2 -f meatware kvmnode2
ccs_tool addnode -n 3 -f meatware kvmnode3
```

将 配置文件(/etc/cluster/cluster.conf)拷贝到另外两台节点上：

在三台节点上启动 cman 程序
```
  service cman start
  service rgmanager start
```
