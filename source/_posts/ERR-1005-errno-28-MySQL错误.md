title: '[ERR] 1005 errno 28 MySQL报错，不能操作元数据'
author: itboyer
categories: database
top_img: /images/top_img/database.jpg
date: 2018-11-27 15:03:58
tags:
  - MySQL
  - Error
---
# 错误定位
公司开发环境的MySQL在使用过程中发生以下错误：
  
![错误截图](/images/pasted-5.png)

经试验发现只要是修改元数据相关操作，alter 和 create操作都会报这个错误

![建表错误](/images/pasted-6.png)

<!-- more -->
在网上搜索相关错误，并结合[MySQL错误code](/MySQL错误errorno大全)表定位到问题：得出结论，磁盘写入 有问题

在服务器上执行`df -h`命令，并没有网上别的帖子中的那种磁盘满了的情况，MySQL的安装目录/data 目录下还有大量的存储空间未使用

![服务器磁盘空间占用](/images/pasted-7.png)

经排查，发现磁盘inodes空间满了，导致不能创建新目录和文件

![inodes使用情况](/images/pasted-8.png)

# 解决方案
删除无用的文件

关于inodes的理解和操作可参考[Free inodes is less than](https://www.jianshu.com/p/c27ee8eeb872)
关于inodes的重新分配可参考[ext4最大inodes文件数](https://www.baishitou.cn/1180.html)