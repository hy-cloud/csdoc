## SR of XenServer

### 1. LVM-Base SR
LVM-Base 的 Sr 主要有 lvm, lvmoiscsi, lvmohba 几种

当虚拟机停止时， 虚拟机磁盘对于的 lvm  被设置为隐藏， 进入/dev/vg-name/目录下，lv 看不到这时可以通过命令：

```
  lvchange -ay /dev/vg-name/lv-name
```
设置 该lv 显示。
