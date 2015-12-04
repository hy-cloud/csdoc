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

## XenServer VM platform Options
```
platform: {
  'pae': 'true',
  'nx': 'true',
  'viridian': 'true',
  'apic': 'true',
  'acpi': 'true'
}
```

  - pae: 物理地址扩展,PAE支持32位处理器可以访问4GB以上物理内存功能版本的Windows，并且它是NX的先决条件。
  - NX: 可让处理器帮助保护电脑免受恶意软件的攻击。
  - APIC:（Advanced Programmable Interrupt Control）高级可编程中断控制器
  - ACPI: Advanced Configuration and Power Interface
  - viridian: 是Windows 的一个特性， 安装windows 是启用该选项

### XenServer HVM 的引导顺序
XenServer HVM 虚拟机使用 BIOS　定义的引导顺序进行引导
boot on floppy (a), hard disk (c), Network (n) or CD-ROM (d)
default: hard disk, cd-rom, floppy
```
  xe vm-param-set HVM-boot-policy="BIOS order" uuid=[uuid of your vm]
```
启动安装完成之后， 更改回来：
```
xe vm-param-set HVM-boot-policy="" uuid=[uuid of your vm]
```

在使用XenAPI 创建虚拟机时， 要指定引导策略和引导顺序
```
vm_rec = {
  HVM_boot_plicy: 'BIOS order',
  HVM_boot_params : { 'order': 'dc'  }
}
```
这样虚拟机就可以从CD-ROM 中启动， 安装虚拟机之后，可以从磁盘启动。
