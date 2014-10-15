##使用pssh进行并行批量操作

假如同时给上千台服务器执行一个命令,拷贝一个文件,杀一个进程等,有什么简化运维管理的工具呢?在小型使用中我都是使用for循环,数量巨大,一方面不确定操作是否成功,一方面for循环语句性能不好估计且是不是同步并行执行.，这类工具比如pdsh，mussh，cssh，dsh等还有这里提到的pssh: 

1.安装:

```
#wget http://peak.telecommunity.com/dist/ez_setup.py
python ez_setup.py
#wget http://parallel-ssh.googlecode.com/files/pssh-2.2.2.tar.gz
# tar zxvf pssh-2.2.2.tar.gz
# cd pssh-2.2.2
# python setup.py install
```

2.pssh使用 (假设ssh已做好SSH信任,ssh信任请参看:关于ssh命令研究以及SSH信任详解) pssh工具包主要有5个程序: 1 pssh  多主机并行运行命令

```
[root@server pssh-2.2.2]# pssh -P -h test.txt uptime
192.168.9.102:  14:04:58 up 26 days, 17:05,  0 users,  load average: 0.07, 0.02, 0.00
192.168.9.102: [1] 14:04:58 [SUCCESS] 192.168.9.102 9922
192.168.8.171:  14:04:59 up 35 days,  2:01,  6 users,  load average: 0.00, 0.00, 0.00
192.168.8.171: [2] 14:04:59 [SUCCESS] 192.168.8.171 22
192.168.9.104:  14:04:59 up 7 days, 20:59,  0 users,  load average: 0.10, 0.04, 0.01
192.168.9.104: [3] 14:04:59 [SUCCESS] 192.168.9.104 9922
[root@server pssh-2.2.2]# cat test.txt
192.168.9.102:9922
192.168.9.104:9922
192.168.8.171:22   //注意我的端口号不仅是默认的22
假如想将输出重定向到一个文件 加-o file 选项
```

2.pscp  把文件并行地复制到多个主机上 注意 是从服务器端给客户端传送文件:

```
[root@server pssh-2.2.2]# pscp -h test.txt /etc/sysconfig/network /tmp/network   //标示将本地的/etc/sysconfig/network传到目标服务器的/tmp/network
```

3.prsync 使用rsync协议从本地计算机同步到远程主机

```
[root@server ~]# pssh -h test.txt -P mkdir /tmp/etc
[root@server ~]# prsync -h test.txt -l dongwm -a -r /etc/sysconfig /tmp/etc //标示将本地的/etc/sysconfig目录递归同步到目标服务器的 /tmp/etc目录下,并保持原来的时间戳,使用用户 dongwm
```

4.pslurp 将文件从远程主机复制到本地,和pscp方向相反:

```
[root@server ~]# pslurp -h test.txt   -L /tmp/test -l root /tmp/network test  //标示将目标服务器的/tmp/network文件复制到本地的/tmp/test目录下,并更名为test
[1] 14:53:54 [SUCCESS] 192.168.9.102 9922
[2] 14:53:54 [SUCCESS] 192.168.9.104 9922
[root@server ~]# ll /tmp/test/192.168.9.10
192.168.9.102/ 192.168.9.104/
[root@server ~]# ll /tmp/test/192.168.9.102/
总计 4.0K
-rw-r--r-- 1 root root 60 2011-04-22 14:53 test
[root@server ~]# ll /tmp/test/192.168.9.104/
总计 4.0K
-rw-r--r-- 1 root root 60 2011-04-22 14:53 test
```

5.pnuke 并行在远程主机杀进程:

```
[root@server ~]# pnuke -h test.txt   syslog //杀死目标服务器的syslog进程,只要ps进程中出现相关词语 都能杀死
[1] 15:05:14 [SUCCESS] 192.168.9.102 9922
[2] 15:05:14 [SUCCESS] 192.168.9.104 9922
```