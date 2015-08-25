# 添加用户说明：
```
// 使用root登陆跳板机（login.123u.com），添加用户脚本路径在/root/EasUseradd.sh
ssh root@login.123u.com
// 格式： bash EasUseradd.sh 域账户名1 域账户名2 域账户名3(可同时添加多个)
bash EasUseradd.sh sharlockqi
```

# 1. 编译安装：
```
tar -zxvf eash-*.tar.gz
cd eash.git\
./configure
make
make install
```


# 2. 安装openssl

`yum -y install openssl`


# 3. 生成证书：

```
cd certs.ok
vim mkcerts	          //将OPENSSL=/usr/bin/openssl后面改为你openssl的执行文件路径
./mkcerts	          //生成证书，期间要输入11次证书密码，要用一样的
cp server.pem client.pem root.pem /etc/eas/certs/	         //将证书复制到程序安装的证书目录，需要覆盖，也可以在命令前面加上yes | 来自动确认
```


# 4. mysql

创建如下两个表单：

```
CREATE TABLE session
(id     integer PRIMARY KEY auto_increment,
 real_uid     INTEGER NOT NULL,
 real_gid     INTEGER NOT NULL,
 effective_uid   INTEGER NOT NULL,
 effective_gid   INTEGER NOT NULL,
 original_uid   INTEGER NOT NULL,
 original_gid   INTEGER NOT NULL,
 port   INTEGER NOT NULL,
 duration     INTEGER NOT NULL,
 real_pw_name   VARCHAR(63) NOT NULL,
 real_gr_name   VARCHAR(63) NOT NULL,
 effective_pw_name      VARCHAR(63) NOT NULL,
 effective_gr_name      VARCHAR(63) NOT NULL,
 original_pw_name       VARCHAR(63) NOT NULL,
 original_gr_name       VARCHAR(63) NOT NULL,
 terminal     VARCHAR(63) NOT NULL,
 ip     VARCHAR(16) NOT NULL,
 status     VARCHAR(63) NOT NULL,
 stype   VARCHAR(63) NOT NULL,
 method     VARCHAR(63) NOT NULL,
 cipher     VARCHAR(63) NOT NULL,
 sysname     VARCHAR(63) NOT NULL,
 nodename     VARCHAR(63) NOT NULL,
 `release`     VARCHAR(63) NOT NULL,
 version     VARCHAR(63) NOT NULL,
 machine     VARCHAR(63) NOT NULL,
 file_session   VARCHAR(63),
 hash_session   VARCHAR(63),
 dns    VARCHAR(127),
 remote_command     VARCHAR(255),
 pid     INTEGER NOT NULL,
 created      DATETIME,
 modified     DATETIME
);

create table data (id  integer PRIMARY KEY auto_increment,
sessionID int NOT NULL,io int NOT NULL,sec int NOT NULL,usec int NOT NULL,len int NOT NULL,data blob);
```


# 5.配置文件

修改easd的数据库部分配置文件，将IP、用户名、密码以及数据库名修正：

`vim /etc/eas/easd_config`

![](manual/programming/easd-install/1.jpg)


# 6. 服务端启动以及开机启动

直接输入easd启动

`vim /etc/rc.d/rc.local`

新增 `/usr/local/sbin/easd` 行，实现开机启动功能；


# 7.客户端启动

修改用户登录shell，vim /etc/passwd 文件，将登陆shell改为eash的执行文件，/usr/local/bin/eash，以便每次用户登录自动启动：

![](manual/programming/easd-install/2.png)

这里提供一个简单的批量添加用户shell，执行时在后面参数直接接上用户名即可，跳板机上添加后直接可用域账户和域密码登录：
```
#!/bin/bash
#Eash user add by qiqi!
for((i=1;i<=$#;i++))
do
useradd -s /usr/local/bin/eash $(eval echo '$'$i)
if [ $? -eq 0 ]
then
        eval echo user '$'$i add successful!
else
        eval echo user '$'$i add failure!
fi
done
```


# 8.注意
1.easd一定要开机启动，数据库也是，否则会造成用户不能正常登录的严重影响，最好留一个对/etc/passwd有写权限的账户，以便出现登录问题时可以用此账户修正过来；如果真的出现登录问题而且没有预备账户的情况下，那就只能在单用户模式下去修正/etc/passwd文件了。

2.如不能使用scp命令，便下载一个openssh的srpm的源码包，将session.c里指定bash即可，重新打包程序，强制、无依赖安装！

例：`rpm -ivh openssh-server-5.3p1-85.4.el6.x86_64.rpm --force --nodeps`

```
              argv[arg_ind++] = (char *)shell;
              argv[arg_ind++] = "-c";
              argv[arg_ind++] = subsystem_path;
              argv[arg_ind++] = NULL;
              execve(shell, argv, env);
              perror(shell);
              exit(254);
            }
```

修改后

```
              argv[arg_ind++] = "/bin/bash";
              argv[arg_ind++] = "-c";
              argv[arg_ind++] = subsystem_path;
              argv[arg_ind++] = NULL;
              execve("/bin/bash", argv, env);
              perror(shell);
              exit(254);
            }
```

**故障处理：**
2015.07.02：
    easd程序异常结束，并且无法启动！
    日志错误：
    Can not find easd.USER table.
    
    这个USER表是sqlite存储在本地的连接记录，经检查都正常。最后发现是/etc/eas/cert/里的1年证书到期，将/root/eash/certs.ok/mkcerts脚本里的所有days配置改为36500，再将/root/eash/certs.ok/conf/目录里的所有证书配置文件默认days也改为36500,再次生成证书并覆盖（详见第3步骤证书生成）。
    再次启动easd，仍然无法启动！
    
    日志错误：Jul  2 19:56:01 13_254_vm_sa_vpn easd[23612]: error: fopen(easd): No such file or directory (2)

    经检查由于之前强制结束掉easd进程，/var/run/easd.pid 内存文件仍然存在，强制删除掉后，正常启动easd。问题解决！
