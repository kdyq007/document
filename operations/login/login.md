## 一、要实现的效果

跳板机外网IP：login.123u.com

登录示意图:

![](manual/programming/login/0.jpg)


*A是客户端，保存私钥。

*B是登录机，只允许使用域帐号密码登录。

*C是内部的服务器，保存公钥。

### 跳板机策略

*各员工使用各自的域密码登录跳板机，然后用自己的私钥，以root用户身份登录到线上服务器。

*除了跳板机，其他服务器的SSH服务不允许对外网开放。

*跳板机禁止用公私钥登录。

*VPN通道将会被逐步禁止使用。



## 二、客户端设置

众多ssh客户端工具中，强烈推荐使用xshell，简单方便；跳板机默认只允许使用帐号密码登录.跳板机外网IP：login.123u.com

### xshell的使用与设置

首先用域账户密码登录跳板机

1.新建连接

![](manual/programming/login/16.jpg)

2.点击确定，并打开刚才新建的连接，输入登录用户名，也就是我们的域帐号（开机帐号）

![](manual/programming/login/17.jpg)

3.输入域密码（开机密码）

![](manual/programming/login/18.jpg)

4.登录跳板机成功

![](manual/programming/login/19.jpg)

下面是通过跳板机ssh到服务器了，请在xshell上做如下设置

1.点击“文件----属性”，选中“SSH”类别，在里面打勾“使用密码处理的Xagent(SSH代理)”：

![](manual/programming/login/1.jpg)


2.点击“工具----Xagent开始”：

![](manual/programming/login/2.jpg)


3.如果之前没有添加过自己的私钥，这里列表会是空白的，点击“键管理”，再在弹出的对话框中点击“导入”，选择我们的私钥文件：

![](manual/programming/login/3.jpg)


4.私钥会自动被添加到Xagent中，但此时私钥还是未启用的，双击我们添加的私钥，输入私钥密码，“确定”：

![](manual/programming/login/4.jpg)


5.私钥变为"open"状态了：

![](manual/programming/login/5.jpg)


6.如图，上一个红框处是没有进行密钥设置的，ssh登录服务器需要输入密码；但进行上述设置后，再次登录就自动用私钥验证了。

![](manual/programming/login/6.jpg)

※注：线上服务器请用root登录	`ssh root@1.1.1.1`



### putty的使用与设置

使用putty想实现以上功能，还需要两个工具：puttygen密钥转换和pageant认证代理，在运行里输入puttygen.exe和pageant.exe启动，如果没有就需要手动去下载了。

1.首先启动puttygen.exe来转换我们的私钥成为putty可用的格式，点击红框中的“load”，选择我们自己的私钥，如果私钥用密码会提示输入，然后点击“Save private key”，填上保存后的名字确定即可，密钥会自动保存为.ppk的格式：

![](manual/programming/login/7.jpg)


2.运行pageant.exe认证代理，点击“Add Key”，选择我们转换过的私钥.ppk，私钥会自动加载到列表框：

![](manual/programming/login/8.jpg)


3.最后打开putty，在“Connection----SSH----Auth”类别的"Authentication forwarding"中把“Allow agent forwarding”选项打勾：

![](manual/programming/login/9.jpg)


4.现在先登录跳板机，再通过跳板机ssh到服务器，成功！

![](manual/programming/login/10.jpg)

※注：线上服务器请用root登录	`ssh root@1.1.1.1`



### secureCRT的使用与设置

secureCRT也支持“agent forwarding”功能，只可惜CRT必须用密钥登录过任意服务器一次后，才能保存。我们可以用一台linux虚拟机搭建sshd服务器放上我们的公钥，然后用私钥登录一次，也可以就在本地搭建OpenSSH for Windows（见附）。

1.首登录跳板机（一定要登录）；

2.对CRT进行如下设置，这里可以选择公钥也可选私钥，但公私钥需要放在一起

![](manual/programming/login/11.jpg)


3.用私钥登录sshd服务器，如果提示输入用户密码点跳过：

![](manual/programming/login/14.jpg)

登录sshd服务器成功：

![](manual/programming/login/12.jpg)


4.如果登录成功，就把跳板机退出一下，重新登录跳板机，这时会在跳板机/tmp目录下生成一个私钥的缓存：

![](manual/programming/login/15.jpg)


5.现在就可以通过跳板机直接用私钥登录其它服务器了：

![](manual/programming/login/13.jpg)

※注：线上服务器请用root登录	`ssh root@1.1.1.1`



### 从本地上传文件到服务器的重要说明
rz、sz命令是终端工具外带的命令，而不是linux系统原有的！经测试，rz命令在通过跳板机向服务器传文件时，由于某些文件是二进制的，如果遇到一些特殊字符会错误的识别为控制字符，处理不当导致传输失败！因此，当遇到传送文件被提前中断的情况时，请在加上`-e`参数，如果传输大文件，最好加上`-b`参数。

```
-b：binary 用binary的方式上传下载，不解释字符为ASCII
-e：强制escape 所有控制字符，比如Ctrl+x，DEL等
```

※注：
1.用rz命令后会弹出上传文件对话框，请不要勾选对话框中“Upload files as ASCII”。
2.在我电脑上测试，同时接`-be`参数可以，但如果接`-eb`，会无效哦！这也是误导了我N长时间的原因。。
