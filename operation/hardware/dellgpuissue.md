## 00：文章简介

GPU服务器在系统下使用nvidia-smi查看信息，可以查看到GPU卡的BUS-id，可以通过BUS-ID和服务器tsr日志中的信息来判断GPu卡的实际槽位

<!-- more -->

## 01：查看GPU的信息

nvidia-smi查看GPU的信息
![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353227842429.jpg)

## 02：查看GPU的BUS-ID

我们可以看见这个红框内的BUS-ID，这个是以16进制显示的，我们换算成10进制

![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353228272152.jpg)

## 03：在tsr上查看video信息

![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353228500308.jpg)

对比tsr收集日志就可以查看到的video信息，Bus:Dev:Func对应的slot槽位，这就是物理位置。

通过上面的对比，我们发现系统下的顺序和slot顺序是不同的，不能使用正常的顺序判断实际槽位。


