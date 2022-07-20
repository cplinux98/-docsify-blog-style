## 00：文章简介

GPU服务器在系统下使用nvidia-smi查看信息，可以查看到GPU卡的BUS-id，可以通过BUS-ID和服务器tsr日志中的信息来判断GPu卡的实际槽位

<!-- more -->

## 01：在线安装cuda驱动

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/cuda-ubuntu1604.pin
mv cuda-ubuntu1604.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/11.1.0/local_installers/cuda-repo-ubuntu1604-11-1-local_11.1.0-455.23.05-1_amd64.deb
dpkg -i cuda-repo-ubuntu1604-11-1-local_11.1.0-455.23.05-1_amd64.deb
apt-key add /var/cuda-repo-ubuntu1604-11-1-local/7fa2af80.pub
apt-get update
sudo apt-get -y install cuda
```

## 02：离线安装cuda驱动

### 2.1：下载驱动
![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353230663140.jpg)

### 2.2：屏蔽nouveau

```bash
编辑blacklist.conf
sudo vi /etc/modprobe.d/blacklist.conf

在文件末尾追加
blacklist nouveau
options nouveau modeset=0

sudo update-initramfs -u

reboot

验证nouveau是否禁用
lsmod | grep nouveau

```

### 2.3：运行二进制包

```bash
sudo chmod a+x  cuda.xx.run
sudo sh cuda.xx.run
```

![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353231935086.jpg)

![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353232014642.jpg)

![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353232073542.jpg)

![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353232138654.jpg)

![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353232204687.jpg)

然后install，安装完成

### 2.4：环境变量

```bash
echo 'export PATH=/usr/local/cuda-11.1/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda-11.1/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
tail -2 ~/.bashrc
source ~/.bashrc
nvcc -V
```

### 2.5：cudnn安装

```bash
tar -zxvf cudnn-11.1-linux-x64-v8.0.5.39.tgz
cp cuda/include/cudnn*.h /usr/local/cuda/include
cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
chmod a+r /usr/local/cuda/include/cudnn*.h /usr/local/cuda/lib64/libcudnn*
```