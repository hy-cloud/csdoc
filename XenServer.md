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

使用模板创建虚拟机时， 是直接从硬盘引导，这个时候设置 HVM_boot_policy 和 HVM_boot_params为上面的值会导致引导失败，系统进入启动画面，但引导失败。
这个时候应按照下面的方式设置：
```
  vm_rec = {
    HVM_boot_policy : '',
    HVM_boot_params: {}
    PV_bootloader: 'pygrub'
  }
```

### XenServer 日志
XenServer 的日志在 /var/log 目录下
- xensource.log xensource.log 日志是由 xapi 记录的
- SMlog : 存储相关操作的日志
- xenstored-access.log: 操作 xenstore 的相关日志
- xcp-rrdd-plugins.log： 性能收集日志


### 导入 vhd 文件到 存储
#### 1. 导入vhd 文件到 NFS, EXT类型的存储
假设是一个Nfs 存储， sruuid=055ac7c8-8860-1bc6-3371-75a0fe0e7870 

NFS 挂载在： /var/run/sr-mount/055ac7c8-8860-1bc6-3371-75a0fe0e7870

vhd 文件： /root/Centos-6.5.vhd

新创建的 vdi 名称为 ROOT-10

  - 首先用 uuidgen 生成一个uuid,  把vhd 文件命名为 uuid.vhd, 把uuid.vhd 文件拷贝到 File-based存储中，执行:
  
    ```
      xe sr-scan uuid=sruuid
    ```
  - 创建一个 vdi, 然后使用dd 命令把 vhd 文件的内容 拷贝的新创建的vdi 对应得文件中
     
    ```
      vsize = vhd-util query -v -n /root/Centos-6.5.vhd
      uuid=$(xe vdi-create sr-uuid=055ac7c8-8860-1bc6-3371-75a0fe0e7870 virtual-size=$vsizeMiB name-label='ROOT-10' type='user')
      
      dd if=$vhdfile of=/var/run/sr-mount/055ac7c8-8860-1bc6-3371-75a0fe0e7870/$uuid bs=2M iflag=direct oflag=direct
    ```
    
#### 2. 导入vhd 文件 到LVM-based 类型的存储，包括： lvm, lvmohba, lvmoscsi
  假设 存储对于的Vg 为 VG_XenStorage-$sruuid
  
  1. 查询vhd 文件的大小， 创建一个vdi
  
    ```
      vsize = vhd-util query -v -n /root/Centos-6.5.vhd
      uuid=$(xe vdi-create sr-uuid=$sruuid virtual-size=$vsizeMiB name-label=ROOT-13 type=user)
      #使用xe vdi-create 创建的vdi, 它对于的lv默认是隐藏的，首先要把它设置为显示，以便可以对该lv进行操作
      lvchange -ay /dev/VG_XenStorage-$sruuuid/VHD-$uuid
    ```
  2. 查询 vdi 的 physical_utilisation 大小
  
    ```
    psize = $(xe vdi-param-get param-name=physical-utilisation uuid=$uuid)
    ```
  3. 拷贝vhd 的内容
  
    ```
      dd if=/root/Centos-6.5.vhd of=/dev/VG_XenStorage-$sruuid/VDH-$uuid bs=2M iflag=direct oflag=direct
    ```
  4. 拷贝vhd 文件开始512个字节，到 vdi 对于的lv physical_utilisation 大小的最后512字节
  
    ```
      dd if=/root/Centos-6.5.vhd of=/dev/VG_XenStorage-$sruuid/VHD-$uuid bs=512 seek=$(( $(($psize / 512)) -1)) count=1
    ```
  5. 设置 lv 的物理利用大小
  
    ```
      vhd-util modify -s $psize -n /dev/VG_XenStorage-$sruuid/VHD-$uuid
    ```
  
