##一、准备篇

###1.配置防火墙，开启80、3306端口

`vim /etc/sysconfig/iptables`

```
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT #允许80端口通过防火墙
-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT #允许3306端口通过防火墙
```

备注：很多网友把这两条规则添加到防火墙配置的最后一行，导致防火墙启动失败，正确的应该是添加到默认的22端口这条规则的下面

如下所示：

添加好之后防火墙规则如下所示

```
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
```

`service iptables restart`	#最后重启防火墙使配置生效

###2.关闭SELINUX

`vim /etc/selinux/config`

```
#SELINUX=enforcing #注释掉
#SELINUXTYPE=targeted #注释掉
SELINUX=disabled #增加
:wq #保存退出
setenforce 0	#实时生效
```

###3.安装第三方yum源

```
wget http://www.atomicorp.com/installers/atomic
sh ./atomic
yum check-update #更新yum源
```


##二、安装篇

###1.安装mysql

```
yum -y install mysql
chkconfig mysqld on
service mysqld restart
```

###2.为root账户设置密码

```
mysqladmin -u root password '123456'	#第一次设置密码
mysqladmin -u root password oldpass '123456'	#修改密码

service mysqld restart

mysql -u root -p	#登录测试
```


###3.安装php以及php组件
```
yum -y install php php-fpm php-mysql php-gd libjpeg* php-imap php-ldap php-odbc php-pear php-xml php-xmlrpc php-mbstring php-mcrypt php-bcmath php-mhash libmcrypt

service php-fpm start
chkconfig php-fpm on
```

###4.安装phpmyadmin
`yum -y install phpmyadmin`



##三、配置篇

###1.apache基础配置

`vim /etc/httpd/conf/httpd.conf #编辑文件`

```
#ServerName www.example.com:80	#在402行	修改为 ServerName localhost:80  这里设置为你自己的域名，如果没有域名，可以设置为localhost

ServerTokens OS　    #在44行  修改为：ServerTokens Prod （在出现错误页的时候不显示服务器操作系统的名称）

ServerSignature On　 #在536行修改为：ServerSignature Off （在错误页中不显示Apache的版本）

Options Indexes FollowSymLinks　 #在331行 修改为：Options Includes ExecCGI FollowSymLinks（允许服务器执行CGI及SSI，禁止列出目录）

#AddHandler cgi-script .cgi　#在796行 修改为：AddHandler cgi-script .cgi .pl （允许扩展名为.pl的CGI脚本运行）

AllowOverride None　 #在338行修改为：AllowOverride All （允许.htaccess）

AddDefaultCharset UTF-8　#在759行 修改为：AddDefaultCharset GB2312　（添加GB2312为默认编码）

Options Indexes MultiViews FollowSymLinks #在554行 修改为 Options MultiViews FollowSymLinks（不在浏览器上显示树状目录结构）

DirectoryIndex index.html index.html.var  #在402行 修改为：DirectoryIndex index.html index.htm Default.html Default.htm index.php Default.php index.html.var  （设置默认首页文件，增加index.php）

KeepAlive Off   #在76行 修改为：KeepAlive On （允许程序性联机）

MaxKeepAliveRequests 100   在83行 修改为：MaxKeepAliveRequests 1000 （增加同时连接数）

:wq!  #保存退出

/etc/init.d/httpd restart #重启
```


###2.PHP配置

`vim  /etc/php.ini   #编辑`

```
date.timezone = PRC     #在946行 把前面的分号去掉，改为date.timezone = PRC

disable_functions = passthru,exec,system,chroot,scandir,chgrp,chown,shell_exec,proc_open,proc_get_status,ini_alter,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru,stream_socket_server,escapeshellcmd,dll,popen,disk_free_space,checkdnsrr,checkdnsrr,getservbyname,getservbyport,disk_total_space,posix_ctermid,posix_get_last_error,posix_getcwd, posix_getegid,posix_geteuid,posix_getgid, posix_getgrgid,posix_getgrnam,posix_getgroups,posix_getlogin,posix_getpgid,posix_getpgrp,posix_getpid, posix_getppid,posix_getpwnam,posix_getpwuid, posix_getrlimit, posix_getsid,posix_getuid,posix_isatty, posix_kill,posix_mkfifo,posix_setegid,posix_seteuid,posix_setgid, posix_setpgid,posix_setsid,posix_setuid,posix_strerror,posix_times,posix_ttyname,posix_uname	#在386行列出PHP可以禁用的函数，如果某些程序需要用到这个函数，可以删除，取消禁用。

expose_php = Off        #在432行禁止显示php版本的信息

magic_quotes_gpc = On   #在745行 打开magic_quotes_gpc来防止SQL注入

open_basedir = .:/tmp/  #在380行设置表示允许访问当前目录(即PHP脚本文件所在之目录)和/tmp/目录,可以防止php木马跨站

:wq!  #保存退出

/etc/init.d/mysqld restart  #重启MySql

/etc/init.d/httpd restart   #重启Apche
```


###3.apache配置虚拟主机

`vim /etc/httpd/conf/httpd.conf #编辑文件`

输入`GG`跳到配置文件末尾

`#NameVirtualHost *:80	#在990 修改为NameVirtualHost www.kdyq.org:80`

复制最后的虚拟主机模版
```
#<VirtualHost *:80>
#    ServerAdmin webmaster@dummy-host.example.com
#    DocumentRoot /www/docs/dummy-host.example.com
#    ServerName dummy-host.example.com
#    ErrorLog logs/dummy-host.example.com-error_log
#    CustomLog logs/dummy-host.example.com-access_log common
#</VirtualHost>
```

修改为如下，以域名或端口来区分：
```
<VirtualHost www.kdyq.org:80>
    ServerAdmin kdyq@vip.qq.com
    DocumentRoot /var/www/html/
    ServerName www.kdyq.org
    ErrorLog logs/www.kdyq.org-error_log
    CustomLog logs/www.kdyq.org-access_log common
</VirtualHost>

<VirtualHost bbs.kdyq.org:80>
    ServerAdmin kdyq@vip.qq.com
    DocumentRoot /var/www/html/bbs
    ServerName bbs.kdyq.org
    ErrorLog logs/bbs.kdyq.org-error_log
    CustomLog logs/bbs.kdyq.org-access_log common
</VirtualHost>
```

###4.apache配置反向代理

检查这几个代理模块是否加载
```
LoadModule rewrite_module modules/mod_rewrite.so
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
```
在需要的地方添加如下两条代理命令（如若没有虚拟目录在配置文件末尾添加即可）：

`ProxyPass /web/ http://localhost:8022/`

`ProxyPassReverse /web/ http://localhost:8022/`
```
<VirtualHost anyterm.kdyq.org:80>
    ServerAdmin kdyq@vip.qq.com
    DocumentRoot /var/www/html/anyterm
    ServerName anyterm.kdyq.org
    ProxyPass / http://bbs.kdyq.org
    ProxyPassReverse / http://bbs.kdyq.org
    ErrorLog logs/anyterm.kdyq.org-error_log
    CustomLog logs/anyterm.kdyq.org-access_log common
</VirtualHost>
```