## 00：文章简介

记录DELL服务器的CentOS系统中安装nvidia显卡驱动。



系统版本：CentOS 7.3

显卡型号：2080TI

驱动版本：Linux



## 01：配置环境

### 1.1：依赖包

```bash
yum -y update
yum -y install kernel-headers firmware kernel-devel gcc* gcc-c++ glibc-* kernel-source kernel-doc
```

### 1.2：关闭通用显卡驱动

```bash
echo -e "blacklist nouveau\noptions nouveau modeset=0" > /etc/modprobe.d/blacklist.conf
 mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r)
 dracut /boot/initramfs-$(uname -r).img $(uname -r)
```

重启检查`lsmod | grep nouveau`，没有显示内容则为成功关闭。

### 1.3：安装dkms


```bash
# 使用yum来安装，自动解决依赖包问题，若不成功，在后面加--force --nodeps
wget http://rpmfind.net/linux/epel/7/ppc64/Packages/d/dkms-2.6.1-1.el7.noarch.rpm
yum -y install dkms-2.6.1-1.el7.noarch.rpm
```



## 02：安装驱动

```
chmod 777 NVIDIA-XXX.run
sh NVIDIA-XXX.run
```

使用下面的命令检查是否安装成功

```bash
lspci | grep NVIDIA
nvidia-smi
```


