## 00：文章简介

记录一下IPMItools的安装和使用方法，这东西用来管理服务器很好用。
Dell还有一个 RACADM 这个工具，对于设置iDRAC很有帮助。

<!-- more -->

## 01：安装

```bash
yum -y install ipmitool

modprobe ipmi_watchdog
modprobe ipmi_poweroff
modprobe ipmi_devintf
modprobe ipmi_msghandler
modprobe ipmi_si
```

本地执行命令直接执行即可，远程执行命令需要添加参数
```bash
ipmitool -I lanplus -H ipaddr -U username -P password
```

## 02：常用命令

### 2.1：BMC相关

```bash
# 冷重启idrac管理卡
ipmitool mc reset cold

# 显示BMC版本信息
ipmitool mc info

# -I 表示interface，open是指OpenIPMI与BMC通信，适用于查看设定本地主机的iDRAC
ipmitool -I open channel info 1

# 设置IP地址
ipmitool lan set 1 ipaddr 192.168.1.2

# 设置子网掩码
ipmitool lan set 1 netmask 255.255.255.0

# 设置网关
ipmitool lan set 1 defgw ipaddr 192.168.1.1

# 创建用户
ipmitool user set name 2 root

# 设置密码
ipmitool user set password 2 calvin

# 开通权限
ipmitool channel setaccess 1 2 callin=on ipi=on link=on privilege=4  //

# 查看chanenel1的用户信息
ipmitool user list 1

```

### 2.2：重启后动作

```bash
# 重启后停在BIOS 菜单
ipmitool chassis bootdev bios
 
# 重启后从PXE启动
ipmitool chassis bootdev pxe　
```

