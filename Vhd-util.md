## vhd-util 的用法

```
  vhd-util COMMAND [ OPTIONS ]
  COMMAND := { create | query | snapshot | read | set | repair | resize | fill | coalesce | modify | scan | check | revert }
```

### create vhd file
```
  vhd-util create  -n name -s size (MB) [ -r reverse ]
  
  -n name : 指定vhd 文件的名称
  -s : 指定 vhd 文件的大小， 单位为 (MB)
  -r : 知道 vhd 文件的格式为 fixed 卷
  
  eg :
  
  vhd-util create -n test.vhd -s 1024
```

###  vhd-util query 
```
  vhd-util query -n name [- v print Virtual Size] [ -s print physical utilization  (bytes) ] [ -p print parent ]
  [-f print fields ] [ -m print marker ] [ -d print chain depth ] [ -S print max virtual size (MB) for  fast resize] [ -h help]
  
  - n : 指定 vhd 文件的名称
  - v : 查询vhd 文件的虚拟大小
  -s : 查询vhd 文件的物理大小
  -p : 查询vhd 文件的 parent
  
  eg : 
  查询 vhd 文件的 虚拟大小:
  
  vhd-util query -n vhd-file -v
  
  查询 vhd 文件的 parent :
  
  vhd-util query -n vhd-file -p
  
  查询 vhd 文件的 物理大小:
  
  vhd-util query -s -n vhd-file
```

### vhd-util snapshot
```
  vhd-util snapshot -n name -p parent-name [ -l snapshot depth limit] [ -m parent_is_raw ]
  为 parent-name 的vhd 文件 创建 名称为 name 的 快照文件
  
  - p : 指定要创建快照的 vhd 文件
  -n :  快照的名称
  -l : 指定快照链的长度限制
  -m : 如果 parent 为 raw 格式时， 要使用 -m 选项
  
  eg : 为 test.vhd 创建快照 snapshot-1.vhd
  
  vhd-util snapshot -n snapshot-1.vhd -p test.vhd
```

### vhd-util coalesce
```
  vhd-util coalesce -n name [-a ancestor ] [-o output] [ -s sparse ] [ -p progress ] [ -h help ]
  把快照卷的数据合并到 parent 卷 中
  
  -o : 把 parent 和快照卷合并， 并保存到 新的vhd文件 中
```

#### vhd-util read
```
  vhd-util read options
  options:
    -h          help
    -n          name
    -p          print VHD headers 打印 VHD文件的头部信息
    -t sec      translate logical sector to VHD location
    -b blk      print bat entry
    -B          print entire bat as a bitmap
    -m blk      print bitmap
    -i sec      test bitmap for logical sector
    -e sec      output extent list of allocated logical sectors
    -a          print batmap
    -j blk      test batmap for block
    -d blk      print data
    -c num      num units
    -r sec      read num sectors at sec
    -R byte     read num bytes at byte
    -x          print in hex

  　eg : print VHD headers
  　
  　vhd-util read -p -n vhd-file
  　VHD Footer Summary:
-------------------
Cookie              : conectix
Features            : (0x00000002) <RESV>
File format version : Major: 1, Minor: 0
Data offset         : 512
Timestamp           : Thu Nov 12 16:42:44 2015
Creator Application : 'tap'
Creator version     : Major: 1, Minor: 3
Creator OS          : Unknown!
Original disk size  : 102400 MB (107374182400 Bytes)
Current disk size   : 102400 MB (107374182400 Bytes)
Geometry            : Cyl: 51400, Hds: 16, Sctrs: 255
                    : = 102398 MB (107372544000 Bytes)
Disk type           : Differencing hard disk
Checksum            : 0xfffff0b4|0xfffff0b4 (Good!)
UUID                : 6b642dab-d7a5-4f45-8827-8d1b211433d0
Saved state         : No
Hidden              : 0

VHD Header Summary:
-------------------
Cookie              : cxsparse
Data offset (unusd) : 18446744073709
Table offset        : 1536
Header version      : 0x00010000
Max BAT size        : 51200
Block size          : 2097152 (2 MB)
Parent name         : 1392d448-4fee-4d3e-bd44-9dfb7fdff63f.vhd
Parent UUID         : 7335a2c4-14fd-438e-acf1-9e23e0e18419
Parent timestamp    : Fri Oct 23 19:00:01 2015
Checksum            : 0xffffd6fa|0xffffd6fa (Good!)

VHD Parent Locators:
--------------------
locator:            : 0
       code         : PLAT_CODE_MACX
       data_space   : 512
       data_length  : 49
       data_offset  : 213504
       decoded name : ./1392d448-4fee-4d3e-bd44-9dfb7fdff63f.vhd

locator:            : 1
       code         : PLAT_CODE_W2KU
       data_space   : 512
       data_length  : 84
       data_offset  : 214016
       decoded name : ./1392d448-4fee-4d3e-bd44-9dfb7fdff63f.vhd

locator:            : 2
       code         : PLAT_CODE_W2RU
       data_space   : 512
       data_length  : 84
       data_offset  : 214528
       decoded name : ./1392d448-4fee-4d3e-bd44-9dfb7fdff63f.vhd

VHD Batmap Summary:
-------------------
Batmap offset       : 206848
Batmap size (secs)  : 13
Batmap version      : 0x00010002
Checksum            : 0xffffffff|0xffffffff (Good!)
```

### vhd-util set
```
  vhd-util set -n name -f field -v value
  为 name vhd 文件 设置 field 属性的值为 value
  
  eg : 设置vhd 文件的 hidden 属性
  
  vhd-util set -n name -f hidden -v 0 | 1 ( 0 : 显示, 1: 隐藏)
```

### vhd-util repair
```
  vhd-util repair -n vhd-file
  修复vhd 文件的 footer 区
```

### vhd-util check
```
  vhd-util check -n vhd-file [ -i ignore missing primary footers ] [ -I ignore parent uuids ] [ -t ignore timestamps ]
  [-B do not check BAT for overlapping ] [ -p check parents ] [ -b check bitmaps] [ -s stats] [ -h help]
  -i :  忽略 丢失 主 footers
  -I :  忽略 parent uuid
  -t :  忽略 时间戳
  -B :  不检查 BAT 重合
  -p :  检查 parents
  -s :  检查 stats
  -b :  检查 bitmaps
```

### vhd-util scan
```
  vhd-util scan [ -m match filter] [ -f fast ] [ -c continue on failure] [ -l LVM volume] [ -p pretty print ] [ -a scan parents ]
  [-v verbose ]
  -m : 扫描指定规则的 vhd 文件
  -p : 格式化输出
  -v : 详细输出
  -l : 扫描 lvm 卷
```

### vhd-util fill
```
  vhd-util fill -n name
  将dynamic类型vhd镜像填满
```

### vhd-util modify
```
  vhd-util modify -n name [ -p new-parent  set parent [-m raw] ] [-s NEW_SIZE set size] [-z zero (kill data)]
   -p : 为 vhd file 指定 parent
   -s : 设置 vhd 文件的大小
   -z : 清空 vhd 文件的数据
```

### vhd-util resize
```
  vhd-util resize -n name  -s size (MB) -j
  因为 resize 过程中，有可能要移动 数据块，resize 操作 必须在 vhd 文件 offline，同时指定 -j 选项。
  如果在创建(vhd-util create/snapshot )的时候， 指定了 -S <msize> 选项， 预先分配了vhd文件增长到 msize 所需要的元数据，
  这种情况下，把vhd文件resize 到 msize 大小，可以直接在线 resize.
  
```

### vhd-util revert
```
  vhd-util revert -n name -j 
```
