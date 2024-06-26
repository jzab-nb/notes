# 一、安装mysql(如已安装可跳过), [参考链接](https://blog.csdn.net/m0_63228448/article/details/121739771)

```
# 1.更新数据源
sudo apt-get update
# 2.安装mysql服务
sudo apt-get install -y  mysql-server
# 3.进行初始化配置
sudo mysql_secure_installation

# 第一步: 是否使用强密码, y使用其他键不使用
# 第二步: 设置密码(如发生错误往下看)
# 第三步: 是否删除默认的匿名用户,可以不删除
# 第四步: 是否禁用远程root访问, 可以不禁用
# 第五步: 是否删除测试数据库, 可以删除
# 第六步: 是否刷新权限表, 可以刷新

# 4.尝试登录mysql, xxxxxx处填自己设置的密码
sudo mysql -uroot -pxxxxxx

```

> 在上面的第二步MYSQL设置密码时显示，Failed! Error: SET PASSWORD has no significance for user ‘root‘@‘localhost‘ as the authe
>
> 出现这种问题使用下述方法解决，[参考链接](https://blog.csdn.net/weixin_42189863/article/details/125113978)

```
# 1.打开一个新的终端

# 2.打开mysql
sudo mysql

# 3.手动修改密码，xxxxx处输入新密码
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password by 'xxxxx';

# 4.重新进行配置
sudo mysql_secure_installation
```

> 安装后一般默认是启动的,可以通过下述的语句来实现服务的开启或关闭

```
/etc/init.d/mysql start  启动
 
/etc/init.d/mysql stop   停止
 
/etc/init.d/mysql restart 重启
 
/etc/init.d/mysql status  查看状态
```

> **远程访问：**
>
> mysql默认启动3306端口, 若使用云服务器应开放3306端口
>
> 同时修改配置文件 /etc/mysql/mysql.conf.d/mysqld.cnf 中的
>
> ​	bind-address =  0.0.0.0
>
> 这一操作可以让mysql服务跑在网络上，然后登录mysql

```
# 进入mysql
sudo mysql -uroot -pxxxxxx

# 使用mysql数据库
> use mysql;
# 查看用户和允许登录主机的情况
> select user,host from user;
# 更新用户 ‘root’ 的允许登录主机为全部
> update user set host = '%' where user = 'root';
# 重新查看
> select user,host from user;
# 退出mysql
> exit

# 重启mysql！！！！！
sudo /etc/init.d/mysql restart

# 查看ip(需要apt安装net-tools包),在第三方工具中连接数据库
ifconfig
```

# 二、安装java并运行

```
# 安装java
sudo apt-get install openjdk-8-jdk

# 测试运行
sudo java -jar jar包名.jar

# ctrl+c退出运行

# 后台运行
sudo nohup java -jar jar包名.jar &
```

# 三、部署常见流程

> 1. 上传文件至指定目录
>
> 2. **sudo netstat -nultp**（需要apt安装net-tools包） 查看端口占用情况
> 3. 若端口被占用使用 **kill -9 进程号** 杀掉占用进程,若未被占用则走下一步
> 4. **sudo** **nohup java -jar jar包名.jar &** 后台启动项目
> 5. 查看项目同级目录下的**nohup.out**文件的输出日志

