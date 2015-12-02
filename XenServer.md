## SR of XenServer

### 1. LVM-Base SR
LVM-Base 的 Sr 主要有 lvm, lvmoiscsi, lvmohba 几种

当虚拟机停止时， 虚拟机磁盘对于的 lvm  被设置为隐藏， 进入/dev/vg-name/目录下，lv 看不到这时可以通过命令：

```
  lvchange -ay /dev/vg-name/lv-name
```
设置 该lv 显示。


## Create VM using CLI

```
uuid=$(xe vm-install template=CentOS\ 6\ \(64-bit\) new-name-label=CentOSNEW)
 vbd-uuid=$(xe vbd-list vm-uuid=${uuid} userdevice=0 params=uuid --minimal)
 
 xe vm-cd-add vm=CentOSNEW cd-name=CentOS-6.5-x86_64-minimal.iso device=3
 
 cid=$(xe vbd-list vm-uuid=${uuid} type=CD params=uuid --minimal)
 
 xe vbd-param-set uuid=${cid} bootable=true
 
 xe vm-param-set uuid=${uuid} other-config:install-repository=cdrom
 
 xe vif-create vm-uuid=${uuid} network-uuid=43347e81-0816-5c8f-c972-8832b5884b0d mac=random device=0
 xe vm-start uuid=${uuid}
```
