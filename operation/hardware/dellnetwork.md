## 00：文章简介

记录一下DELL 服务器更新网卡固件的方法。

<!-- more -->


## 01：下载网卡固件

在Dell官网的“支持-->产品支持-->输入对应服务器型号-->输入关键字-->选择操作系统-->下载使用驱动“。

![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353205223362.jpg)



## 02：下载完成后放在U盘中

### 2.1：更改驱动名称为简化名称

例如：1或按驱动版本命名

#### 2.1.1：在linux系统下（需下载.bin的二进制文件）

```bash
mount /dev/u盘盘符 /mnt	#挂载U盘，可通过"lsblk"来查看U盘盘符
cp 下载的固件 /root		#将固件复制到/root路径下
cd /root				#切换到/root路径
umount /dev/u盘盘符	#卸载U盘挂载
./下载的固件    #安装固件
reboot             #完成后重启
```

#### 2.1.2：在Windows系统下（需下载.exe的可执行文件）

```bash
以管理员身份执行.exe格式的文件
点击install（安装）并按照提示选择安装
重启
```

#### 2.1.3：在Lifecycle Controller（生命周期控制器）下（需下载微软原生文件）

```bash
开机按F10，进入Lifecycle Controller下
在左侧找到固件更新（Firmware Update）
在主窗格中单击启动固件更新（Launch Firmware Update）
在选择更新存储库（Update Repository）页面，选择本地驱动器（ Local Drive），然后点击下一步（Next）
在访问详细信息（Access Details）页面，从本地驱动器下拉列表中选择可移动USB存储
在文件路径或更新包路径（path and filename of the firmware）文本框中，输入更新包的名字例如：1.exe
点击下一步（Next）
在选择更新页面中，勾选刚才输入的安装包名字，然后点击应用（Apply）
等待更新完成
```

## 03：验证固件版本是否更新成功

1. 在idrac网卡界面，查看网卡固件版本
2. 在linux系统下，使用ethtool -i p1p1 查看固件版本
3. 在BIOS界面查看
