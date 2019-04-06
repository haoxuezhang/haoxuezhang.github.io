---
layout: post
title: Ubuntu下配置mysql远程访问以及防火墙设置
categories: mysql
description: Ubuntu下配置mysql远程访问以及防火墙设置
keywords: Ubuntu, mysql, 防火墙, 远程访问
---

远程访问Linux服务器中mysql的配置方法

---


## mysql配置远程访问
配置文件修改，`vim /etc/mysql/my.cnf`,找到 “bind-address = 127.0.0.1” , 这一行要注释掉，只需在前面加个#，即 # bind-address = 127.0.0.1
然后修改mysql系统权限
```sql
mysql -u root -p
password : 你的密码
mysql>use mysql;
mysql>select 'host' from user where user='root';
mysql>update user set host = '%' where user ='root';
mysql>flush privileges;
```
再重新启动MySQL
```sh
service mysql restart
```
最后也是最重要的一步，阿里云的服务器设置了安全组规则来限制ecs服务器的ip,端口访问策略。因此需要修改。

登录阿里云=>控制台=>云服务器ECS=>网络和安全=>安全组

在入方向，点击配置规则可以看到下图，3306端口是访问服务器mysql的，没有的话就添加规则，端口范围选择 3306mysql,授权对象设置为0.0.0.0/0 允许所有ip访问。80端口是访问web的，22端口是远程连接服务器的。

## 防火墙配置
```sh
sudo apt-get install iptables
```
开放端口，根据需求更改端口号
``` sh
iptables -I INPUT -p tcp --dport 80 -j ACCEPT
```
最后保存规则
```sh
iptables-save
```
但是这样配置不是永久配置，如果服务器重启，防火墙配置会还原，这里我们需要在安装一下工具来帮我们实现持久化，这里我们使用 iptables-persistent
```sh
sudo apt-get install iptables-persistent
```
```sh
sudo netfilter-persistent save
sudo netfilter-persistent reload
```
最后可以通过命令查看哪些端口是开放的
```sh
netstat -nupl (UDP类型的端口)
netstat -ntpl (TCP类型的端口)
```
完成
