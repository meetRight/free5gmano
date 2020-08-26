## free5gmano安装注意问题
1.别名定义命令无法使用，注意后续命令操作；<br>

2.设置数据库环境变量，注意修改root用户的密码，参见后续教程<br>

```
1)	输出安装时自动生成的debian.cnf文件内容
sudo cat /etc/mysql/debian.cnf
输出内容如下
# Automatically generated for Debian scripts. DO NOT TOUCH!
[client]
host     = localhost
user     = debian-sys-maint
password = iqhZ4BsjJvWsGXfy
socket   = /var/run/mysqld/mysqld.sock
[mysql_upgrade]
host     = localhost
user     = debian-sys-maint
password = iqhZ4BsjJvWsGXfy
socket   = /var/run/mysqld/mysqld.sock
2)	使用文件中的user段的debian-sys-maint用户进行登录
$ mysql -udebian-sys-maint -p
Enter password: // 这里输入上面文件内的password段的值
3)	修改root用户的插件和密码
此处一定要记得改plugin的值为mysql_native_password
UPDATE mysql.user SET plugin="mysql_native_password", authentication_string=PASSWORD("password") WHERE user="root";
如果上一步骤中出现密码过于简单无法修改的情况，可以先将validate_password_policy的值设置为0或者LOW。
SET GLOBAL validate_password_policy=0;
4)	重启mysql就可以使用新设置的密码进行登录了
sudo service mysql restart
```

3.使用插件时需要先修改free5gmano-cli/nm/setting.py里面的修改端口号为8000；<br>

4.注册插件需要使用对应的tacker-example-plugin插件，注册插件之前需要先修改tacker-example-plugin/config.yaml的配置参数，主要是nm和nfvo的IP地址；<br>

5.网络功能和网络切片的模板需要更换；<br>


## 附录
1.	安全组<br>
>使用默认的安全组即可，需要在其中添加两个管理规则，一个是SSH，一个是全部ICMP的即可。<br>

2.	Ubuntu18.04配置DNS<br>
```
最近使用了最新版的ubuntu 18.04运行一些服务，然后发现服务器经常出现网络不通的情况，主要是一些域名无法解析。
检查/etc/resolv.conf，发现之前修改的nameserver总是会被修改为127.0.0.53，无论是改成啥，过段时间，总会变回来。
查看/etc/resolv.conf这个文件的注释，发现开头就写着这么一行：
# This file is managed by man:systemd-resolved(8). Do not edit.
这说明这个文件是被systemd-resolved这个服务托管的。
通过netstat -tnpl | grep systemd-resolved查看到这个服务是监听在53号端口上。
查了下，这个服务的配置文件为/etc/systemd/resolved.conf，大致内容如下：
[Resolve]
DNS=
#FallbackDNS=
#Domains=
#LLMNR=no
#MulticastDNS=no
#DNSSEC=no
#Cache=yes
#DNSStubListener=yes
如果我们要想让/etc/resolve.conf文件里的配置生效，需要添加到systemd-resolved的这个配置文件里DNS配置项（如上面的示例，已经完成修改），然后重启systemd-resolved服务即可。
另一种更简单的办法是，我们直接停掉systemd-resolved服务，这样再修改/etc/resolve.conf就可以一直生效了。
```
3.	使用cinder删除卷(https://www.jianshu.com/p/b9fadf973494)<br>
