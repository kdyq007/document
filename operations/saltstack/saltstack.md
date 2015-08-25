# saltstack安装与使用说明

### 一、salt的简介

Saltstack是一个具备puppet与func功能为一身的集中化，轻量级的自动化运维管理工具，使用python编写，功能非常强大,可以使用EPEL快速安装。相比较puppet，安装和配置更加容易和简单。下面是Saltstack安装和基础配置文档。（官方文档：[http://docs.saltstack.com/topics/installation/rhel.html](http://docs.saltstack.com/topics/installation/rhel.html)）

### 二、salt的安装

1. 准备工作
- CentOS7,salt-master,192.168.12.169;
- CentOS6,salt-minion1,192.168.12.186;
- CentOS5,salt-minion2,192.168.12.187;

2. 安装salt

```
分别在各台电脑上导入EPEL YUM源,并更新
wget http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm

rpm -ivh epel-release-6-8.noarch.rpm

yum update

安装服务端：
yum -y install salt-master

安装客户端
yum -y install salt-minion

日志查看路径：（有问题可查日志获取出错信息）
服务端：/var/log/salt/master
客户端：/var/log/salt/minion
```

3. 服务端配置salt-master

默认情况下，salt master在所有接口(0.0.0.0)上监听4505和4506两个端口. 如果想bind某个具体的IP，需要对/etc/salt/master配置文件中"interface"选项做如下修改:

`interface: 192.168.12.169`

`注：192.168.12.169 是本机服务端的IP地址`

修改auto_accept为True，自动接受客户端的KEY，当然也可以这里不设置，手动接受就行，接受方式：salt-key -a keyname （keyname即为客户端刚才设置的id标识）

`auto_accept: True`

```
service salt-master start

systemctl enable salt-master.service (centos7开机启动)
```

4. 客户端配置salt-minion
需要修改minion的配置文件/etc/salt/minion中的master选项，进行如下操作:

master: 192.168.12.169

id :68

注：192.168.12.169 是服务端的IP地址

id :客户端的标识，用服务端连接时，就是用此标识来连接客户端，如：salt '68' cmd.run 'df -h'，如果不填则是默认的主机名

```
service salt-minion start

systemctl enable salt-minion.service
```

5. Master与Minion认证

- minion在第一次启动时，会在/etc/salt/pki/minion/（该路径在/etc/salt/minion里面设置）下自动生成minion.pem(private key)和minion.pub(public key)，然后将minion.pub发送给master。

- master 在接收到minion的public key后，通过salt-key命令accept minion public key，这样在master的/etc/salt/pki/master/minions下的将会存放以minion id命名的public key, 然后master就能对minion发送指令了。(由于前面设置了auto_accept为真，所以这里可以跳过这步)。

6. Master与Minion的连接

Saltstack master启动后默认监听4505和4506两个端口。4505(publish_port)为salt的消息发布系统，4506(ret_port)为salt客户端与服务端通信的端口。如果使用lsof查看4505端口，会发现所有的Minion在4505端口持续保持在ESTABLISHED

`lsof -i :4505`

### 三、KEY管理

Salt在master和minion数据交换过程中使用AES加密, 为了保证发送给minion的指令不会被篡改，master和minion之间认证采用信任的接受(trusted, accepted )的key.

在发送命令到minion之前，minion的key需要先被master所接受(accepted). 运行salt-key可以列出当前key的状态

```
[root@salt /]#salt-key -L
Accepted Keys:
230
68
Unaccepted Keys:
Rejected Keys:
```

注：
Accepted Keys为被服务端接受的KEY（230，68这二台客户端是被服务端接受了的KEY，其实230，68就是minion中的id标识号）

Unaccepted Keys:未被服务端接受的KEY

Rejected Keys：被服务端拒绝的KEY

salt-key命令可以接受特定的单个key或批量接受key, 使用-A选项接受当前所有的key, 接受单个key可以使用-a keyname.

常用的认证命令有如下：

```
-a ACCEPT, --accept=ACCEPTAccept the following key
-A, --accept-all    Accept all pending keys

-r REJECT, --reject=REJECTReject the specified public key

-R, --reject-all    Reject all pending keys

-d DELETE, --delete=DELETEDelete the named key

-D, --delete-all    Delete all keys
```

### 四、发送指令

master和minion之间可以通过运行test.ping远程命令判断是否存活：

```
[root@drfdai-17 src]# salt -E '230|68' test.ping
230:
	True
68:
	True
```

或者对所有minion进行：salt  '*' test.ping，返回True说明测试是OK的，客户端是存活状态。

### 五、执行命令
```
salt '68' cmd.run 'df -h'

salt -E '230|68' cmd.run 'df -h'
```

注：把客户端id和发送的命令，用单引号括起来，养成习惯，防止出错

### 六、在服务端salt匹配minion id
在运行salt命令进行匹配时，请使用单引号，避免shell解析

```
匹配所有minion:salt  '*' test.ping

匹配下边域的所有minion:salt '*.example.*' test.ping

匹配example.net域中的(web1.example.net、web2.example.net......webN.example.example.net):salt 'web?.example.net' test.ping

匹配web1到web5的minion: salt 'web[1-5]' test.ping

匹配web-x、web-y及web-z minion: salt 'web-[x-z]' test.ping
```

- 正则表达式
匹配web-prod和web1-devel minion:
`salt -E 'web1-(prod|devel)' test.ping`

- 指定列表
`salt -L 'web1,web2,web3' test.ping`

- 指定组
在服务务端中打开master配置文件

vim /etc/salt/master

添加如下分组
```
nodegroups:
group1: 'L@230,68'
group2: '68'
group3: 'G@os:centos'
group4: 'G@mem:487'
```
值得注意的是编辑master的时候，group1和group2前面是2个空格

- 测试

```
[root@salt /]#salt -N group2 test.ping
68:
True
[root@salt /]# salt -N group1 test.ping
230:
True
68:
True
```

可能大家会好奇group1中为什么会有L@，这代表什么意思？
其实L是指客户端列表，我们一组中有多个客户端，所以在前面用L表示。
除了有列表匹配外，还有很多匹配方式，如：

这些参数都可以直接在命令行使用,如：

salt -S '192.168.1.230' test.ping

salt -G 'os:Centos' test.ping

salt -L '230,68' test.ping

### 七、minion基本信息的管理

- Salt基本命令介绍:
```
salt '*' grains.ls  查看grains分类
salt '*' grains.items 查看grains所有信息
salt '*' grains.item osrelease 查看grains某个信息
```

如：
```
[root@salt /]# salt '*' grains.item osrelease
230:
	osrelease: 7.0.1406
68:
	osrelease: 7.0.1406
```

Saltstack执行远程shell命令，使用cmd.run。如：

`salt '68' cmd.run 'df -h'`

- 内置执行模块

官方模块地址：http://docs.saltstack.com/ref/modules/all/index.html
