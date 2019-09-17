#   Ansible常用模块详解

#### 命令模块

command
shell

#### 文件模块

copy
fetch
file

#### 安装模块

yum

#### 服务模块

service

#### 挂载模块

mount

#### 定时任务

cron

#### 用户模块

user

group

ansible内置了丰富的模块供用户使用，但是经常使用到的模块却不多。本文主要记录了ansible的一些常用模块以及详细参数 、注意事项等 ，供大家学习。

## 命令模块

#### command

概要
命令模块 适合使用简单的命令 无法支持"<"，">"，"|"，";"，"&"等符号
官方文档：https://docs.ansible.com/ansible/latest/modules/command_module.html#command-module

参数		     释义
chdir		    在执行命令前，进入到指定目录中
creates		判断指定文件是否存在，如果存在，不执行后面的操作
removes		判断指定文件是否存在，如果存在，执行后面的操作
free_form		必须要输入一个合理的命令

备注:无法支持"<"，">"，"|"，";"，"&"等符号

示例:

```
root@m01 ~]# ansible dkaiyun -m command -a "hostname"

web01 | CHANGED | rc=0 >>
web01

nfs01 | CHANGED | rc=0 >>
nfs01

backup01 | CHANGED | rc=0 >>
backup
```

chdir

```
[root@m01 ~]# ansible dkaiyun -m command -a "chdir=/data ls -l"
web01 | CHANGED | rc=0 >>
total 4
-rw-r--r-- 1 root root 158 Jan 12 11:11 hosts

backup01 | CHANGED | rc=0 >>
total 4
-rw-r--r-- 1 root root 4 Jan 13 18:06 lol.txt

nfs01 | CHANGED | rc=0 >>
total 4
-rw-r--r-- 1 root root 13 Jan 17 18:45 bbb.txt
```

creates

```
[root@m01 ~]# ansible dkaiyun -m command -a "touch /data/lol.txt creates=/data/lol.txt"
 [WARNING]: Consider using the file module with state=touch rather than running touch.  If you need to use command because file is insufficient you can add
warn=False to this command task or set command_warnings=False in ansible.cfg to get rid of this message.
nfs01 | CHANGED | rc=0 >>
backup01 | SUCCESS | rc=0 >>
skipped, since /data/lol.txt exists
web01 | CHANGED | rc=0 >>
```

removes

```
[root@m01 ~]# ansible dkaiyun -m command -a "rm -f /data/hosts removes=/data/hosts"
nfs01 | SUCCESS | rc=0 >>
skipped, since /data/hosts does not exist

backup01 | SUCCESS | rc=0 >>
skipped, since /data/hosts does not exist

 [WARNING]: Consider using the file module with state=absent rather than running rm.  If you need to use command because file is insufficient you can add
warn=False to this command task or set command_warnings=False in ansible.cfg to get rid of this message.

web01 | CHANGED | rc=0 >>
```

#### shell

概要
类似command模块升级版—万能模块
官方文档：https://docs.ansible.com/ansible/latest/modules/shell_module.html#shell-module

参数		 释义
chdir		在执行命令前，进入到指定目录中
creates		判断指定文件是否存在，如果存在，不执行后面的操作
removes		判断指定文件是否存在，如果存在，执行后面的操作
free_form		必须要输入一个合理的命令

备注：可以使用"<"，">"，"|"，";"，"&"等符号特殊符号

示例:

```
[root@m01 ~]# ansible dkaiyun -m shell -a "ps -ef |grep /[s]sh"
backup01 | CHANGED | rc=0 >>
root       2042      1  0 09:06 ?        00:00:00 /usr/sbin/sshd -D
nfs01 | CHANGED | rc=0 >>
root       1258      1  0 08:32 ?        00:00:00 /usr/sbin/sshd -D
web01 | CHANGED | rc=0 >>
root       1197      1  0 11:39 ?        00:00:00 /usr/sbin/sshd -D
```

注：其它参数参考command模块 使用方法一致

## 文件模块

#### copy

概要
主要用于将管理主机上的数据信息传送给多台主机
官方文档：https://docs.ansible.com/ansible/latest/modules/copy_module.html#copy-module
参数	选项/默认值	   释义
src		                           指定将本地管理主机的什么数据信息进行远程复制
backup	no* yes	      默认数据复制到远程主机，会覆盖原有文件（yes 将源文件进行备份）
content		                  在文件中添加信息
dest（required）		 将数据复制到远程节点的路径信息
group		                      文件数据复制到远程主机，设置文件属组用户信息
mode		                      文件数据复制到远程主机，设置数据的权限 eg 0644 0755
owner		                     文件数据复制到远程主机，设置文件属主用户信息
remote_src	no* yes	如果设置为yes，表示将远程主机上的数据进行移动操作如果设置为no， 表示将管理主机上的数据进行分发操作

备注 （required）为必须使用的参数
*为默认参数
copy模块在复制数据时，如果数据为软链接文件，会将链接指定源文件进行复制
修改权限时候 需要加0 例如：chmod 0644 0755
示例:
```
[root@m01 ~]# ansible web01 -m copy -a "src=./anaconda-ks.cfg  dest=/data"
web01 | CHANGED => {
    "changed": true, 
    "checksum": "9d791df2961e299fac1206c2e1c6ab1cde2c86a2", 
    "dest": "/data/anaconda-ks.cfg", 
    "gid": 0, 
    "group": "root", 
    "md5sum": "221e5656c9b59aec6c7596568fff8ad3", 
    "mode": "0644", 
    "owner": "root", 
    "size": 1499, 
    "src": "/root/.ansible/tmp/ansible-tmp-1548229670.84-2879942383233/source", 
    "state": "file", 
    "uid": 0
}
```
backup
```
[root@m01 ~]# ansible web01 -m copy -a "src=./anaconda-ks.cfg  dest=/data backup=yes"
web01 | CHANGED => {
    "backup_file": "/data/anaconda-ks.cfg.4263.2019-01-23@15:52:43~", 
    "changed": true, 
    "checksum": "9d791df2961e299fac1206c2e1c6ab1cde2c86a2", 
    "dest": "/data/anaconda-ks.cfg", 
    "gid": 0, 
    "group": "root", 
    "md5sum": "221e5656c9b59aec6c7596568fff8ad3", 
    "mode": "0644", 
    "owner": "root", 
    "size": 1499, 
    "src": "/root/.ansible/tmp/ansible-tmp-1548229931.86-180942706957431/source", 
    "state": "file", 
    "uid": 0
}
```
```
[root@web01 ~]# ll /data/
total 8
-rw-r--r-- 1 root root 1499 Jan 23 15:52 anaconda-ks.cfg
-rw-r--r-- 1 root root 1505 Jan 23 15:52 anaconda-ks.cfg.4263.2019-01-23@15:52:43~
```
owner group mode
```
[root@m01 ~]# ansible web01 -m copy -a "src=./anaconda-ks.cfg  dest=/data owner=www group=www mode=0644"
web01 | CHANGED => {
    "changed": true, 
    "checksum": "9d791df2961e299fac1206c2e1c6ab1cde2c86a2", 
    "dest": "/data/anaconda-ks.cfg", 
    "gid": 1086, 
    "group": "www", 
    "md5sum": "221e5656c9b59aec6c7596568fff8ad3", 
    "mode": "0644", 
    "owner": "www", 
    "size": 1499, 
    "src": "/root/.ansible/tmp/ansible-tmp-1548230667.08-106764271060692/source", 
    "state": "file", 
    "uid": 1086
}
```
```
[root@web01 data]# ll
total 4
-rw-r--r-- 1 www www 1499 Jan 23 16:04 anaconda-ks.cfg
```
content
```
[root@m01 ~]# ansible web01 -m copy -a "content=test  dest=/data/anaconda-ks.cfg "
web01 | CHANGED => {
    "changed": true, 
    "checksum": "a94a8fe5ccb19ba61c4c0873d391e987982fbbd3", 
    "dest": "/data/anaconda-ks.cfg", 
    "gid": 1086, 
    "group": "www", 
    "md5sum": "098f6bcd4621d373cade4e832627b4f6", 
    "mode": "0644", 
    "owner": "www", 
    "size": 4, 
    "src": "/root/.ansible/tmp/ansible-tmp-1548231000.52-150895010308573/source", 
    "state": "file", 
    "uid": 1086
}
```
```
[root@web01 data]# cat anaconda-ks.cfg 
test[root@web01 data]#
注：content添加内容不会添加回车符 
```
#### fetch
概要
抓取文件到管理机上
官方文档：https://docs.ansible.com/ansible/latest/modules/fetch_module.html#fetch-module
参数		                    释义
src（required）		要获取的远程系统上的文件，必须是文件，而不是目录
dest		                    用于保存文件的目录
示例:
```
[root@m01 ~]# ansible web01 -m fetch -a "src=/root/lol.txt dest=/root"
web01 | CHANGED => {
    "changed": true, 
    "checksum": "da39a3ee5e6b4b0d3255bfef95601890afd80709", 
    "dest": "/root/web01/root/lol.txt", 
    "md5sum": "d41d8cd98f00b204e9800998ecf8427e", 
    "remote_checksum": "da39a3ee5e6b4b0d3255bfef95601890afd80709", 
    "remote_md5sum": null
}
[root@m01 ~]# tree ~
/root
└── web01
    └── root
        └── lol.txt
2 directories, 1 file
[root@m01 ~]# 
```
#### file
概要
实现创建/删除文件信息 对数据权限进行修改
官方文档：https://docs.ansible.com/ansible/latest/modules/file_module.html#file-module
参数	                                               选项/默认值	释义
dest/path/name（required）		                       将数据复制到远程节点的路径信息
group		                                                                文件数据复制到远程主机，设置文件属组用户信息
mode		                                                                 文件数据复制到远程主机，设置数据的权限 eg 0644 0755
owner		                                                                文件数据复制到远程主机，设置文件属主用户信息
src		                                                                      指定将本地管理主机的什么数据信息进行远程复制
state	                                                absent	      将数据进行删除
=	                                                       directory	  创建一个空目录信息
=	                                                       file	             查看指定目录信息是否存在
=	                                                       touch	        创建一个空文件信息
=	                                                       hard/link	  创建链接文件
备注
示例:
```
[root@m01 ~]# ansible web01 -m file -a "src=/etc/fstab dest=/tmp/fstab state=link"
[root@m01 ~]# ansible web01 -m file -a "path=/tmp/fstab state=absent"
[root@m01 ~]# ansible web01 -m file -a "path=/tmp/test state=touch"
```


## 安装模块

#### yum

概要
使用yum软件包管理器安装，升级，降级，删除和列出软件包和组。
官方文档：https://docs.ansible.com/ansible/latest/modules/yum_repository_module.html#yum-repository-module
参数	                      选项/默认值	            释义
name（required）		                             指定软件名称信息
state	                     absent/removed	   将软件进行卸载（慎用）
=	                            present/installed	  将软件进行安装
latest		                                                     安装最新的软件 yum update

备注

示例:

```
[root@m01 ~]# ansible web01 -m yum -a "name=httpd-tools state=installed"
[root@m01 ~]# ansible web01 -m yum -a 'name=httpd state=latest'
[root@m01 ~]# ansible web01 -m yum -a 'name="@Development tools" state=present'
[root@m01 ~]# ansible web01 -m yum -a 'name=http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm state=present'
```

## 服务模块

#### service

概要
用于管理服务运行状态
官方文档：https://docs.ansible.com/ansible/latest/modules/service_module.html#service-module
参数	               选项/默认值	             释义
enabled	        no yes	                      设置服务是否开机自启动 如果参数不指定，原有服务开机自启动状态进行保留
name （required）		                      设置要启动/停止服务名称
state=	            reloaded	                 平滑重启
=	                     restarted	                 重启
=	                     started	                     启动
=	                     stopped	                   停止
备注
示例:

```
[root@m01 ~]# ansible web01 -m service -a "name=crond state=started enabled=yes"
```

## 挂载模块

#### mount

概要
用于批量管理主机进行挂载卸载操作
官方文档：https://docs.ansible.com/ansible/latest/modules/mount_module.html#mount-module
参数	选项/默认值	释义
fstype		                 指定挂载的文件系统类型
opts		                    指定挂载的参数信息
path		                    定义一个挂载点信息
src		                       定义设备文件信息
state	absent	       会进行卸载，也会修改fstab文件信息
=	       unmounted 会进行卸载，不会修改fstab文件
=	       present	    不会挂载，只会修改fstab文件
=	       mounted	 会进行挂载，会修改fstab文件

在进行挂载的时候，使用state=mounted
在进行卸载的时候，使用state=absent

示例:

```
[root@m01 ~]# ansible web01 -m mount -a "src=172.16.1.31:/data/  path=/mnt fstype=nfs state=present"
以上信息只是在/etc/fstab文件中添加了配置信息，不会真正进行挂载（mount -a）
[root@m01 ~]# ansible web01 -m mount -a "src=172.16.1.31:/data/  path=/mnt fstype=nfs state=mounted"
以上信息是在/etc/fstab文件中添加了配置信息，并且也会真正进行挂载
```



## 定时任务

#### cron

概要
定时任务模块
官方文档：https://docs.ansible.com/ansible/latest/modules/cron_module.html#cron-module

参数	                                                         选项/默认值	释义
minute/hour/day/month/weekday		                        和设置时间信息相关参数
job		                                                                               和设置定时任务相关参数
name（required）		                                                   设置定时任务注释信息
state	                                                        absent	       删除指定定时任务
disabled	                                                  yes	             将指定定时任务进行注释
=	                                                               no	              取消注释

备注：时间参数不写时，默认为 *

示例:
每五分钟同步一次时间

```
[root@m01 ~]# ansible web01 -m cron -a "name='ntpdate time' minute=*/5 job='/usr/sbin/ntpdate ntp1.aliyun.com &>/dev/null' "
web01 | CHANGED => {
    "changed": true, 
    "envs": [], 
    "jobs": [
        "None", 
        "ntpdate time"
    ]
}
```


结果

```
[root@web01 data]# crontab -l
# Ansible: ntpdate time
*/5 * * * * /usr/sbin/ntpdate ntp1.aliyun.com &>/dev/null
```


删除定时任务

```
[root@m01 ~]# ansible web01 -m cron -a "name='ntpdate time' state=absent"
web01 | CHANGED => {
    "changed": true, 
    "envs": [], 
    "jobs": []
}
```


注释定时任务
注意：注释和取消注释时必须填写 job和时间 参数

```
[root@m01 ~]# ansible web01 -m cron -a "name='ntpdate time' minute=*/5 job='/usr/sbin/ntpdate ntp1.aliyun.com &>/dev/null' disabled=yes"
web01 | CHANGED => {
    "changed": true, 
    "envs": [], 
    "jobs": [
        "ntpdate time"
    ]
}
```

结果

```
[root@web01 data]# crontab -l
# Ansible: ntpdate time
# */5 * * * * /usr/sbin/ntpdate ntp1.aliyun.com &>/dev/null
```

取消注释

```
[root@m01 ~]# ansible web01 -m cron -a "name='ntpdate time' minute=*/5 job='/usr/sbin/ntpdate ntp1.aliyun.com &>/dev/null' disabled=no"
web01 | CHANGED => {
    "changed": true, 
    "envs": [], 
    "jobs": [
        "ntpdate time"
    ]
}
```

结果

```
[root@web01 data]# crontab -l
# Ansible: ntpdate time
*/5 * * * * /usr/sbin/ntpdate ntp1.aliyun.com &>/dev/null
```

## 用户模块

#### user

概要
远程批量创建用户信息
官方文档：https://docs.ansible.com/ansible/latest/modules/user_module.html#user-module

参数	              选项/默认值	    释义
password		                             请输入密码信息
name		                                    指定用户名信息
uid		                                        指定用户uid信息
group		                                   指定用户主要属于哪个组
groups		                                 指定用户属于哪个附加组信息
shell	              /bin/bash或/sbin/nologin	   指定是否能够登录
create_home	yes/no	          是否创建家目录信息
home		                                   指定家目录创建在什么路径 默认/home
备注：password设置密码时不能使用明文方式，只能使用密文方式
可以给用户设置密码 还可以给用户修改密码

示例:

```
ansible db -m user -a 'name="testops" password="$6$0lwTSmqKOkL.ktgl$OnBexXC7haBf0FRHVMIZM2edDeFWBbpKJ2r9cxVwNvY.vh3IIUzwFz8n7jFglc0CrtQSY12ziDonVL6e71Og2."'
```

```
ansible db -m user -a 'name="testops" state="absent" remove="yes"'
```

#### group

概要
远程批量创建用户组信息
官方文档：https://docs.ansible.com/ansible/latest/modules/group_module.html#group-module

参数	选项/默认值	释义
gid		                      指创建的组ID信息
name		                 指创建组名称信息
state	absent	      删除指定的用户组
=	       present	    创建指定的用户组
备注

示例:
创建一个指定的用户组dkaiyun gid=1055

```
ansible web01 -m group -a "name=dkaiyun gid=1055"
```


删除一个指定的用户组dkaiyun gid=1055

```
ansible web01 -m group -a "dkaiyun gid=1055 state=absent"
```



