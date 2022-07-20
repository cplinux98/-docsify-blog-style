## 00：文章简介

记录了查看网卡IP的几种方法，以及系统硬件信息的查询方法。


<!-- more -->

## 01：查看网卡IP的几种方法

```bash
ifconfig eth0|grep 'inetaddr'|awk -F '[ :]' '{print $13}'
ifconfig eth0|sed -n '2p'|awk-F '[ :]+' '{print $4}'
ifconfig eth0 |awk -F'[ :]+' 'NR==2 {print $4}'
ifconfig eth0 |sed -nr '2s#^.*dr:(.*)  B.*$#\1#gp'
ifconfig eth0|grep 'inet addr'|awk -F ":" '{print $2}'|awk '{print $1}'
ifconfig eth0|grep 'inetaddr'|cut -d ":" -f2|cut -d " " -f1
```

## 02：Linux查看硬件信息

```
主板的序列号：dmidecode | grep -i 'serial number'
CPU信息：cat /proc/cpuinfo 或者 dmesg | grep -i 'cpu' 或者 dmidecode -t processor
内存信息：cat /proc/meminfo
显卡信息：lspci |grep -i 'VGA' 或者 dmesg | grep -i 'VGA'
网卡信息：lspci | grep -i 'eth'
PCI信息：lspci
键盘和鼠标：cat /proc/bus/input/devices
各设备的中断请求（IRQ）：cat /proc/interrupts
硬件信息，包括bios、cpu、内存等信息：dmidecode
系统配置：/usr/platform/sun4u/sbin/prtdiag –v
硬件汇总信息：dmesg | more
显示外设信息， 如usb，网卡等信息：lspci
已加载的驱动：lsmod 或者 lshw
机器的产品名称及序列号：cat /sys/class/dmi/id/product_*
机器的BIOS信息：cat /sys/class/dmi/id/bios_*
DMA信息：cat /proc/dma
支持的文件系统：cat /proc/filesystems
```

## 03：获取当前目录下占用空间最大的5个文件或目录

```bash
cd /
du -hsx * | sort -rh | head -5

1.1T /backup
876G /data
3.5G /usr
1.3G /var
1.3G /root
```