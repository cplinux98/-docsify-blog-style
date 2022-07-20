## 00：文章简介

保存关于ssh的脚本

## 01：远程连接统计脚本

```bash
[root@centos7 data]# ss -nt | tr -s " " : | cut -d: -f6 |tr -d "[:alpha:]" | uniq -c | sort -nr
      3 192.168.3.5
      1 192.168.3.231
      1 
```