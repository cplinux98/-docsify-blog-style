## 00：文章简介

整理了一下之前买的R420服务器相关配置文档，设置风扇速度后可实现固定风扇速度，来达到“静音”的效果。

<!-- more -->


## 01：开启远程


### 1.1：老版本idrac

java6

### 1.2：2.30.30.30版本以上idrac

可以使用html模式

（建议使用2.30.30.30，新版本强制开启高速风扇）

![image-20210622110622128](https://image.lichunpeng.cn/linux98.com/images20210622111103.png)

## 02：风扇调速

### 2.1：下载windows版ipmitools

```
https://dl.dell.com/FOLDER04161746M/1/OM-BMC-Dell-Web-WIN-8.5.0-2372_A00.exe
```

### 2.2：开启ipmi远程管理

![image-20210622110648571](https://image.lichunpeng.cn/linux98.com/images20210622111115.png)

### 2.3：开启或关闭自动调速

```powershell
#开启自动调速
"C:\Program Files (x86)\Dell\SysMgt\bmc\ipmitool.exe" -I lanplus -H IP地址 -U 账户 -P 密码 raw 0x30 0x30 0x01 0x01
#关闭自动调速
"C:\Program Files (x86)\Dell\SysMgt\bmc\ipmitool.exe" -I lanplus -H IP地址 -U 账户 -P 密码 raw 0x30 0x30 0x01 0x00
```

### 2.4：关闭自动调速，并设置风扇速度

```powershell
#关闭自动调速
"C:\Program Files (x86)\Dell\SysMgt\bmc\ipmitool.exe" -I lanplus -H IP地址 -U 账户 -P 密码 raw 0x30 0x30 0x01 0x00
#设置为0x0c 12% 这个值是16进制的，10% 0x0a | 13% 0x0d
"C:\Program Files (x86)\Dell\SysMgt\bmc\ipmitool.exe" -I lanplus -H IP地址 -U 账户 -P 密码 raw 0x30 0x30 0x02 0xff 0x0c 
```

### 2.5：查看速度

![image-20210622110735097](https://image.lichunpeng.cn/linux98.com/images20210622111120.png)

### 2.6：创建快捷方式

创建两个.bat文本文件，将下面内容写入

```powershell
#开启自动调速.bat
"C:\Program Files (x86)\Dell\SysMgt\bmc\ipmitool.exe" -I lanplus -H IP地址 -U 账户 -P 密码 raw 0x30 0x30 0x01 0x01

#设置速度为13%.bat
"C:\Program Files (x86)\Dell\SysMgt\bmc\ipmitool.exe" -I lanplus -H IP地址 -U 账户 -P 密码 raw 0x30 0x30 0x01 0x00
"C:\Program Files (x86)\Dell\SysMgt\bmc\ipmitool.exe" -I lanplus -H IP地址 -U 账户 -P 密码 raw 0x30 0x30 0x02 0xff 0x0d 
```

