## CloudStack 管理 XenServer 时 SR 的 heartbeat

当XenServer 存储中中一个主机连接不到存储的时候，卷在该存储上的虚拟机不能正常工作：
对于Windows 虚拟机， 或出现蓝屏
对于Linux 虚拟机， 文件系统变成Read-only
在这种情况下，应该关闭所以虚拟机，以免虚拟机出现损害。

CloudStack 在管理 XenServer 时，使用一种 heartbeat 的机制来检测主机连接存储池是否正常，为每一个主机在存储上创建一个 hb-hostuuid 的 vdi,

对于NFS 存储， vdi 为：
  /var/run/sr-mount/sr-uuid/hb-hostuuid.vhd
  
对于基于 LVM 的存储, vdi 为:
 /dev/VG_XenStorage/VHD-hostuuid
 
在每个主机上运行 一个 xenheartbeat.sh 的脚本来检测存储挂载的目录或LVM VG, 以及往 主机对应的vdi 中写 当前的时间，
当在指定的时间内，没有检测到存储目录，或不能往对于的vdi 中写数据，就认为该主机不能连接存储，执行重启host

### 1. 检测存储已经挂载
检测 保存在 /opt/cloud/bin/heartbeat 文件中记录的目录

```
   对于NFS， 检测 /var/run/sr-mount/sr-uuid 目录是否存在
   
   对于 LVM-Based 的存储， 检测 /dev/VG_XenStorage 是否存在
```

如果 不存在， 在/var/log/messages 日志中记录：

```
  Problem with heartbeat, no iSCSI or NFS mount defined in $file!
```

### 2. 往 heartbeat-vdi 写时间数据:

#### 1. NFS 存储：
```
  date +%s | dd of=$hb count=100 bs=1 2>/dev/null
```

#### 2. 对于 LVM-Based 存储

```
  date +%s | dd of=$hb count=100 bs=1 2>/dev/null
```

如果不存在，记录日志:
```
  Potential problem with $hb: not reachable since  XXX seconds
```

### 3. 如果 超时，重启主机
```
Problem with $hb: not reachable for XX second    s, rebooting system!"

```

### 4. 故障案例：

 当存储 为 NFS 时， 如果在 NFS 服务器上执行 Copy 等操作， 会使 xenheartbeat.sh 写 heartbeat-vdi 失败，导致heartbeat 超时，
 从而导致主机重启。
 
 解决方案：
 注释脚本的 reboot -f 命令， 重新运行该脚本.
 
 日志记录在 /var/log/messages 文件中， 以 heartbeat 开头，当主机重启时，我们可以先检测 是否是由 heartbeat 存储时引起的。
