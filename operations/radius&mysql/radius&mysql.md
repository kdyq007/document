# radius连接mysql认证

### 一、radius的简介

radius，远端用户拨入验证服务（RADIUS, Remote Authentication Dial In User Service）是一个AAA协议，意思就是同时兼顾验证（authentication）、授权（authorization）及计费（accounting）三种服务的一种网络传输协议（protocol），通常用于网络存取、或流动IP服务，适用于局域网及漫游服务。

### 二、radius的安装

1. 准备工作
- CentOS7，radius服务端;
- mysql用户账户数据库；
- radius客户端，可以在linux里安装，也可以直接使用windows里的客户端，我这里使用的是windows里的一款绿色工具NTRadPing.exe

2. 安装salt-master

yum install freeradius freeradius-mysql freeradius-utils -y

![](manual/programming/radius&mysql/1.jpg)

低版本centosyum源里可能没有freeradius的安装包，可以去官网下载源码包安装（http://http://freeradius.org/）

3. 编辑freeradius配置文件，开启mysql认证

- 文件1：`/etc/raddb/radiusd.conf`，搜索sql.conf（700行），去掉前面的注释，表示包含sql.conf文件；

- 文件2：`/etc/raddb/sql.conf`，如下图，database是指数据库类型,默认mysql不用管；server是数据库ip地址,login是radius登录上验证的用户名,password是密码，`radius_db`是要给radius新建的数据库名，这些都可以修改，下面还有一些选项是表单名，一般不需要修改。

![](manual/programming/radius&mysql/1.jpg)


打开从数据库查询nas支持
`sed -i 's/\#readclients/readclients/g' /etc/raddb/sql.conf`

- 文件3:`/etc/raddb/sql/mysql/admin.sql`,修改默认的radius连接数据库密码（如果上面的密码有修改的话）

	`sed -i 's/radpass/newpassword/g' /etc/raddb/sql/mysql/admin.sql`


- 文件4: `/etc/raddb/sql/mysql/dialup.conf`,打开在线人数查询支持

 查找`simul_count_query`将279-282行的4个#全去掉。
 

- 文件5：`/etc/raddb/clients.conf`，客户端的配置，在文件底部添加下面一段。只允许下面列出的IP连接radius认证，像我们这次的实验，只需要填写路由器IP就可以了，只允许路由器认证（ipaddr是允许认证的IP，netmask是子网掩码，secret是认证密钥，最好用一个乱组的字符串，因为一般不会修改例：dsf34$jhg$12@#）：
```
以IP：
client DD-WRT1 {
                      ipaddr          = 192.168.12.185
                      secret          = testing123
                      shortname       = DD-WRT1
              }
或
以网段：
client DD-WRT2 {
                      ipaddr          = 192.168.12.0
                      netmask         = 24
                      secret          = testing123
                      shortname       = DD-WRT1
              }
```

- 文件6：`/etc/raddb/sites-available/default`，修改`/etc/raddb/sites-enabled/default`也一样，它是链接的sites-available目录里的。
```
找到authorize {}模块，注释掉files（170行，以文件验证用户），去掉sql前的#号（177行，启用sql验证）
找到accounting {}模块，注释掉radutmp(396行，取消数据跟踪),注释掉去掉sql前面的#号(395行)。
找到session {}模块，注释掉radutmp（450行），去掉sql前面的#号（454行）。
找到post-auth {}模块，去掉sql前的#号（475行 和 563行）。
```

- 文件7：`/etc/raddb/sites-available/inner-tunnel`，同上。
```
找到authorize {}模块，注释掉files（124行），去掉sql前的#号（131行）。
找到session {}模块，注释掉radutmp（251行），去掉sql前面的#号（255行）。
找到post-auth {}模块，去掉sql前的#号（277行 和 301行）。
```


4. 创建radius数据库和导入表
- 创建数据库（123456是你数据库的root密码）：
`mysqladmin -h 10.0.128.220 -uroot -p123456 create radius;`

- 导入表：
```
mysql -h 10.0.128.220 -uroot -p123456 < admin.sql
mysql -h 10.0.128.220 -uroot -p123465 radius < ippool.sql
mysql -h 10.0.128.220 -uroot -p123456 radius < schema.sql
mysql -h 10.0.128.220 -uroot -p123456 radius < wimax.sql
mysql -h 10.0.128.220 -uroot -p123456 radius < cui.sql
mysql -h 10.0.128.220 -uroot -p123456 radius < nas.sql
```

- 导入完后需要使用这样一条命令来新建radius用户，并且让其能够远程查询,'radius'@'192.168.12.186',前面是用户名，后面是允许登录的IP地址，也就是我们的radius服务器IP。
`grant all privileges on *.* to 'radius'@'192.168.12.186' identified by '123456' with grant option;`

![](manual/programming/radius&mysql/5.jpg)


5. 在数据库里添加验证用户，这里提供一个脚本RadiusUserAdd.sh，差不多就这样insert进数据库，可以自己写程序，添加加密。
```
#!/bin/bash
HOSTNAME="10.0.128.220"
PORT="3306"
USERNAME="root"
read -s -p "Enter the passwd of root is:" p
PASSWD="$p"
echo
DBNAME="radius"
TABLENAME="radcheck"
read -p "username is:" user
read -p "password is:" pwd
insert_sql="insert into $DBNAME.${TABLENAME}(Username,Attribute,op,Value) values('$user','password','==','$pwd')"
mysql -h${HOSTNAME} -P${PORT} -u${USERNAME} -p${PASSWD} ${DBNAME} -e "${insert_sql}"
if [ $? == 0 ];then
         echo "Add the pptp_user is ok"
else
         echo "inserting into mysql is not ok, why?"
fi
```

执行效果：

![](manual/programming/radius&mysql/3.jpg)

![](manual/programming/radius&mysql/4.jpg)


6. 至此radius服务器是搭建完毕，启动服务成功。
```
service radiusd start	启动服务
chkconfig radiusd on	设置为开机启动
```

7. 验证，用前文说的windows上的测试工具NTRadPing.exe。

![](manual/programming/radius&mysql/6.jpg)

![](manual/programming/radius&mysql/7.jpg)

### 实验完成