&emsp;&emsp;`NetBeans`由Sun公司（2009年被甲骨文收购）在2000年创立，它是开放源运动以及开发人员和客户社区的家园，旨在构建世界级的Java IDE。NetBeans当前可以在Solaris、Windows、Linux和Macintosh OS X平台上进行开发，并在SPL(Sun公用许可)范围内使用。NetBeans IDE已经支持PHP、Ruby、JavaScript、Groovy、Grails和C/C++等开发语言。      &emsp;&emsp;个人觉得NetBeans最大的好处就是能实现各种跨平台编程的远程部署，下面我们就来看看C语言远程调试环境的配置吧！


## 一、远程编译器的添加与配置


&emsp;&emsp;首先，我们需要一台已经安装好了的linux主机，用作调试器，当然，linux系统编程开发的组件包必须得安装，，例如：gcc、g+\+等，还有SSH必须得开启，一般默认都是开启了的。我现在有一台linux的虚拟主机，IP为192.168.12.115，保持网络的畅通性，以免影响代码的编译调试。

![](manual/programming/Netbeans/1.jpg)




&emsp;&emsp;其次，我们安装好NetBeans,现在就来新建一个C语言项目看看：


![](manual/programming/Netbeans/2.jpg)




&emsp;&emsp;项目的创建都大同小异，下面我们来看看如何添加远程主机为编译器呢：


![](manual/programming/Netbeans/3.jpg)




&emsp;&emsp;依次点击NetBeans菜单栏上的"工具"----“选项”----"C/C+\+"----"编辑",如图：


![](manual/programming/Netbeans/4.jpg)




&emsp;&emsp;再点击“添加”，弹出主机选择框，在主机名处填入我们的linux主机IP或者主机名，当然也可以在下面的主机列表中选择：


![](manual/programming/Netbeans/5.jpg)




&emsp;&emsp;下一步，填入登录用户名，这里我用root，其他用户可能需要给予不同目录、文件的读取访问权限，下一步输入密码，勾选保存密码，继续下一步就已经成功连接到了linux主机，并且列出了linux主机的GNU开发工具集合，我们选中为默认并确定：


![](manual/programming/Netbeans/6.jpg)




&emsp;&emsp;程序返回到了NetBeans的选项界面，在构建主机的下拉框中选择我们刚才添加的linux主机，如图：


![](manual/programming/Netbeans/7.jpg)





&emsp;&emsp;但现在我们可以看到代码前端还有红色的错误图标，提示找不到头文件，因为还需要在项目属性上设置使用linux的编译器来编译。在左边项目名上“右键”----“属性”：


![](manual/programming/Netbeans/8.jpg)




&emsp;&emsp;在构建主机上选择linux的主机，再来看看代码，已经全部成功了，可以执行或调试，Netbeans自动将代码上传至linux主机并执行/调试，下面的输出栏成功输出了“Hello Sharlockqi!“的字段：


![](manual/programming/Netbeans/9.jpg)      &emsp;&emsp;





## 二、自定义头文件与库文件路径的添加


&emsp;&emsp;上面的例子项目是一句最简单的C语言代码，但如果有大量的代码和调用了其它自定义头文件、库文件，该怎样让NetBeans找到它们呢？下面我们来添加一段连接mysql的C语言代码：


![](manual/programming/Netbeans/10.jpg)




&emsp;&emsp;可以看到所有与mysql头文件有关的东西都出错找不到。那么我们来添加mysql的头文件以及库文件调用路径。依然是打开NetBeans菜单栏上的"工具"----“选项”----"C/C+\+"----"代码帮助",注意上面的工具集合一定要是linux主机的,然后在包含目录的列表框右边点击"添加",弹出一个目录查找对话框，默认在读取windows的C盘目录，但我们现在连接的是linux，所以它呈现假死状态，我们点击一下右上角的“向上一级按钮”，就刷出了linux的根目录，选择mysql的头文件目录“/usr/include/mysql”:


![](manual/programming/Netbeans/11.jpg)





&emsp;&emsp;此时，包含目录列表框里就多出了我们添加的目录，编译时会自动寻找这里，点击“确定”。


![](manual/programming/Netbeans/12.jpg)





&emsp;&emsp;但代码还是提示错误，同样，我们还需要修改项目中的路径，打开项目属性选中“构建”下的“C编译器”，在“常规”标签中的“包含目录和头文件”里，直接填上linux文件系统的头文件路径，这里选择会弹出windows的文件系统，所以直接手写吧！


![](manual/programming/Netbeans/13.jpg)




&emsp;&emsp;同时，我们再点击“构建”里面的“链接器”，在“其他库目录”右边同样以手动填写的方式写下mysql的库目录，值得注意的是如果系统是64位，mysql的库路径是`/usr/lib64/mysql`,而如果系统是32位的话，库路径就是`/usr/lib/mysql`:


![](manual/programming/Netbeans/14.jpg)




&emsp;&emsp;最后我们点击下面“库”标签右边的按钮，点击右边列表的“添加选项”，这是针对有些程序编译需要带有独特的参数选项来定制的，我们添加`-lmysqlclient`的编译选项：


![](manual/programming/Netbeans/15.jpg)

![](manual/programming/Netbeans/16.jpg)




&emsp;&emsp;最后连续三次确定返回NesBeans主面板，但此时代码还提示有错误存在，不过现在只是程序没有刷新而已了，你可以在项目上右键----“构建”来更新它，或者直接执行或调试。如图是我已经开始正常下断点并单步调试了。：


![](manual/programming/Netbeans/17.jpg)


## 文档结束，谢谢观看！