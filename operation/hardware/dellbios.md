## 00：文章简介

记录一下DELL 服务器BIOS驱动更新的方法。

<!-- more -->


## 01：下载bios驱动程序

首先下载需要更新的bios驱动程序，保存到一个文件夹中，我们选择bios2.7版本
![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353210102176.jpg)

## 02：使用iDRAC的方式升级

### 2.1：重新启动服务器，检查BIOS当前版本为2.2.2

![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353210645088.jpg)

### 2.2：上传固件

登录iDRAC，在"iDRAC设置→更新和回滚"中，在"固件"选项卡选择"更新"，浏览选择下载的BIOS升级文件（文件名为BIOS_戴尔更新包为原生微软64位格式），单击"上载"按钮。

![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353211052234.jpg)

### 2.3：更新并重启

上传完成后，在"更新详细信息"中显示了上传的BIOS更新文件的状态及版本号，选中上传的更新文件，单击"安装并重新引导"，将会立即安装更新并故事引导服务器。
![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353211356879.jpg)

### 2.4：确认警告

此时会弹出"系统警报"，单击"确定"按钮，之后重新启动服务器，进入BIOS更新任务应用程序。

![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353211736589.jpg)

### 2.5：更新BIOS完成

更新BIOS完成后，系统会重新启动，此时可以看到BIOS版本已经升级。
![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353212003022.jpg)


## 03：使用window的方式升级

### 3.1：下载

将下载的文件保存到任何位置（不能使用双字节路径），双击并运行下载的文件，然后单击Install（安装）。

![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353212558004.jpg)

### 3.1：执行

![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353212685363.jpg)


检查要安装的版本，上面会列出了要更新的硬盘驱动器，如果列表没有问题，则单击Yes（是）。
![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353212838722.jpg)

### 3.2：更新

应用更新可能需要几分钟时间，如果没有响应，请等待一小时左右。

![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353213003509.jpg)

### 3.3：完成

出现“The update completed successfully”（更新已成功完成）消息时，单击OK（确定）。

单击Finish（完成）以完成更新，必要时重新启动系统。至此，固件更新过程便已正常完成。

在OpenManage Server Administrator等工具上检查固件是否已更新。

![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353213216082.jpg)

## 04：使用linux的方式升级。

### 4.1：插入U盘

首先先进入系统，然后将下载好的Bin二进制文件放到U盘里，插进服务器
![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353213998860.jpg)

### 4.2：系统内执行
将U盘插入服务器后，进入系统挂载U盘，挂载U盘的命令是mount  /dev/sdb4/(U盘的盘符，用fdisk -l 就能看到)  /mnt 
![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353214078482.jpg)

我们进入mnt，看到要更新的文件包，然后将其复制到系统里，这里我们将这个文件复制到临时目录/~下
![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353214212485.jpg)
复制到/~下就好办了，我们直接执行 ./xxx.Bin  即可，因为它是二进制软件包，我们直接./就可以执行，系统扫描后会询问你  yes|no，输入yes就开始更新了，等他更新完成就会显示successful   然后重启一下就行了
![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353214329466.jpg)

![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353214405748.jpg)

### 4.3：执行过程
![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353214489499.jpg)

### 4.4：完成
![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353214565055.jpg)


## 05：使用U盘的方式升级

### 5.1：插入U盘

首先把bios2.7拷入U盘，然后插入服务器的USB口上
![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353214830537.jpg)

### 5.2：重启进入F12

然后启动戴尔720服务器，按F12进入bios设置界面，进去之后点击固件更新，然后启动固件更新。
![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353214973833.jpg)
更新第一步，择本地驱动器，点击下一步。
![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353215092243.jpg)
然后在路径上面输入下载的文件名，点击下一步。
![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353215233503.jpg)

### 5.3：执行更新

接着出现了bios当前版本和可用版本，然后点击应用。
![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353215373955.jpg)

### 5.4：完成

系统就开始更新，然后更新完之后，我们检查当前版本为2.7
![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353215522695.jpg)

