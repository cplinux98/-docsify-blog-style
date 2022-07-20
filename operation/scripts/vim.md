## 00：文章简介

在工作中，我们经常会重复的使用很多命令去维护服务器，如果我们把这些重复的命令，写成一个个的shell脚本，就会大大的提升我们的工作效率。


<!-- more -->

下面是一些工作中会遇到的一些脚本。

## 01：vimrc

> 有的时候写了很久的脚本，我们很容易忘记这个脚本是谁写的了，或者这个脚本是干什么的了，所以我们编辑一下.vimrc这个文件，让我们vim创建的.sh脚本都自动带有创建时间和创建人，这样也方便我们后期进行脚本的管理。

设置方法：在用户的家目录下，创建一个.vimrc的文件，然后将下面的内容粘贴进去，退出当前远程exit重进即可生效。

```bash
[root@centos7 ~]# pwd
/root
[root@centos7 ~]# vim .vimrc
```

.vimrc的内容

```bash
set ignorecase
set cursorline
set autoindent
autocmd BufNewFile *.sh exec ":call SetTitle()"
func SetTitle()
	if expand("%:e") == 'sh'
	call setline(1,"#!/bin/bash") 
	call setline(2,"#") 
	call setline(3,"#********************************************************************") 
	call setline(4,"#Author:		lichunpeng") 
	call setline(5,"#QQ: 			492739915") 
	call setline(6,"#Date: 			".strftime("%Y-%m-%d"))
	call setline(7,"#FileName：		".expand("%"))
	call setline(8,"#URL: 			http://www.lichunpeng.cn")
	call setline(9,"#Description：		The test script") 
	call setline(10,"#Copyright (C): 	".strftime("%Y")." All rights reserved")
	call setline(11,"#********************************************************************") 
	call setline(12,"") 
	endif
endfunc
autocmd BufNewFile * normal G
```

效果

vim xxx.sh的效果

```bash
#!/bin/bash
#
#********************************************************************
#Author:                lichunpeng
#QQ:                    492739915
#Date:                  2019-10-29
#FileName：             xxx.sh
#URL:                   http://www.lichunpeng.cn
#Description：          The test script
#Copyright (C):         2019 All rights reserved
#********************************************************************
                                                                                                                     
~
```