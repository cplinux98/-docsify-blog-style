## 00：文章简介

保存管理用户的一些脚本

## 01：用户统计脚本

统计出/etc/passwd文件中其默认shell为非/sbin/nologin的用户个数，并将用户都显示出来

```bash
#!/bin/bash
#
#********************************************************************
#Author:                lichunpeng
#QQ:                    492739915
#Date:                  2019-10-29
#FileName：             usershell.sh
#URL:                   http://www.lichunpeng.cn
#Description：          The test script
#Copyright (C):         2019 All rights reserved
#********************************************************************
COL="\033[$[RANDOM%7+31]m"
LOR="\033[0m"

User_Number=`grep -v "/sbin/nologin" /etc/passwd | wc -l`
User_List=`grep -v "/sbin/nologin" /etc/passwd | cut -d: -f1`

echo -e "您系统下默认shell为非/sbin/nologin的用户个数为：$COL $User_Number $LOR "
echo -e "这些用户为：\n$COL$User_List $LOR"    
```

效果展示：
![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353336416631.jpg)

> 多次执行脚本会变颜色哟！

## 02：UID统计脚本

查出用户UID最大值的用户名、UID及shell类型
```bash
[root@centos7 data]# cat useruid.sh 
#!/bin/bash
#
#********************************************************************
#Author:		lichunpeng
#QQ: 			492739915
#Date: 			2019-10-29
#FileName：		useruid.sh
#URL: 			http://www.lichunpeng.cn
#Description：		The test script
#Copyright (C): 	2019 All rights reserved
#********************************************************************
COL="\033[$[RANDOM%7+31]m"
LOR="\033[0m"

Max_User_UID=`cat /etc/passwd | cut -d: -f1,3,7 | sort -t: -k2 -nr | head -n 1|cut -d: -f2`
Max_User_Name=`cat /etc/passwd | cut -d: -f1,3,7 | sort -t: -k2 -nr | head -n 1|cut -d: -f1`
Max_User_Shell=`cat /etc/passwd | cut -d: -f1,3,7 | sort -t: -k2 -nr | head -n 1|cut -d: -f3`

echo -e "您系统下UID最大的用户名为：$COL$Max_User_Name$LOR"
echo -e "您系统下UID最大的用户的UID为：$COL$Max_User_UID$LOR"
echo -e "您系统下UID最大的用户默认shell类型为：$COL$Max_User_Shell$LOR"
```

效果演示
![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353337032052.jpg)

## 03：创建用户脚本

功能:使用一个用户名做为参数，如果指定参数的用户存在，就显示其存在，否则添加之;显示添加的用户的id号等信息

```bash
#!/bin/bash
#
#********************************************************************
#Author:		lichunpeng
#QQ: 			492739915
#Date: 			2019-10-29
#FileName：		createuser.sh
#URL: 			http://www.lichunpeng.cn
#Description：		The test script
#Copyright (C): 	2019 All rights reserved
#********************************************************************
COL="\033[$[RANDOM%7+31]m"
LOR="\033[0m"

[ 0 -eq "$#" ] && { echo "Usage: `basename $0` USERNAME" ; exit 10;}
id $1 &> /dev/null && { echo -e "用户$COL $1 $LOR 已经存在！" ; exit 20;}
useradd $1 &> /dev/null && echo -e "$COL $1 $LOR 用户已经创建成功！" ; id $1 || echo "出现了一些错误!" ; exit 30
```

效果展示

![](https://image.lichunpeng.cn/mweb-linux98/2021/10/27/16353337837000.jpg)