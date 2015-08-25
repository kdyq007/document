一、编译和链接
	大家都知道，通常我们在写好源代码后，需要先编译，将每一个源码文件编译生成.o或.obj的中间文件；再链接，将源码和需要的头文件链接起来，并生成执行文件。windows中都有IDE为我们自动完成，而在linux中，就需要我们自己完成了，下面来看看Makefile的介绍。
    
二、Makefile介绍

通常我们安装一个源码包，首先执行./configure;然后是make;最后就是make install了。

1../configure是用来检测你的安装平台的目标特征的。比如它会检测你是不是有CC或GCC，并不是需要CC或GCC，它是个shell脚本;

2.make是用来编译的，它从Makefile中读取指令，然后编译;

3.make install是用来安装的，它也从Makefile中读取指令，安装到指定的位置。

下面我们就来看看makefile的基本格式：
Makefile的规则

在讲述这个Makefile之前，还是让我们先来粗略地看一看Makefile的规则。

```
    target ... : prerequisites ...
            command
            ...
            ...

    target也就是一个目标文件，可以是Object File，也可以是执行文件。还可以是一个标签（Label），对于标签这种特性，在后续的“伪目标”章节中会有叙述。

    prerequisites就是，要生成那个target所需要的文件或是目标。

    command也就是make需要执行的命令。（任意的Shell命令）
```

这是一个文件的依赖关系，也就是说，target这一个或多个的目标文件依赖于prerequisites中的文件，其生成规则定义在command中。说白一点就是说，prerequisites中如果有一个以上的文件比target文件要新的话，command所定义的命令就会被执行。这就是Makefile的规则。也就是Makefile中最核心的内容。

#一个makefile的例子
```
 edit : main.o kbd.o command.o display.o /
           insert.o search.o files.o utils.o
            cc -o edit main.o kbd.o command.o display.o /
                       insert.o search.o files.o utils.o

    main.o : main.c defs.h
            cc -c main.c
    kbd.o : kbd.c defs.h command.h
            cc -c kbd.c
    command.o : command.c defs.h command.h
            cc -c command.c
    display.o : display.c defs.h buffer.h
            cc -c display.c
    insert.o : insert.c defs.h buffer.h
            cc -c insert.c
    search.o : search.c defs.h buffer.h
            cc -c search.c
    files.o : files.c defs.h buffer.h command.h
            cc -c files.c
    utils.o : utils.c defs.h
            cc -c utils.c
    clean :
            rm edit main.o kbd.o command.o display.o /
               insert.o search.o files.o utils.o
```

反斜杠（/）是换行符的意思。这样比较便于Makefile的易读。我们可以把这个内容保存在文件为“Makefile”或“makefile”的文件中，然后在该目录下直接输入命令“make”就可以生成执行文件edit。如果要删除执行文件和所有的中间目标文件，那么，只要简单地执行一下“make clean”就可以了。

在这个makefile中，目标文件（target）包含：执行文件edit和中间目标文件（*.o），依赖文件（prerequisites）就是冒号后面的那些 .c 文件和 .h文件。每一个 .o 文件都有一组依赖文件，而这些 .o 文件又是执行文件 edit 的依赖文件。依赖关系的实质上就是说明了目标文件是由哪些文件生成的，换言之，目标文件是哪些文件更新的。

在定义好依赖关系后，后续的那一行定义了如何生成目标文件的操作系统命令，一定要以一个Tab键作为开头。记住，make并不管命令是怎么工作的，他只管执行所定义的命令。make会比较targets文件和prerequisites文件的修改日期，如果prerequisites文件的日期要比targets文件的日期要新，或者target不存在的话，那么，make就会执行后续定义的命令。

这里要说明一点的是，clean不是一个文件，它只不过是一个动作名字，有点像C语言中的lable一样，其冒号后什么也没有，那么，make就不会自动去找文件的依赖性，也就不会自动执行其后所定义的命令。要执行其后的命令，就要在make命令后明显得指出这个lable的名字。这样的方法非常有用，我们可以在一个makefile中定义不用的编译或是和编译无关的命令，比如程序的打包，程序的备份，等等。
    
三、使用automake自动生成makefile文件

通过这张图，我们可以很好的理解makefile文件自动生成的过程：

![](manual/programming/makefile and automake/1.gif)

1.新建一个hello目录，并编写一个hellodb.c文件,其中调用了mysql.h头文件，代码如下：

```
#include <stdio.h>
#include <mysql.h>

int main(void)
{
        printf("hello mysql!!!\n");
        return 0;
}
```

2.输入autoscan命令，生成configure.scan文件，将它重命名为configure.in：

![](manual/programming/makefile and automake/2.jpg)

3.修改configure.in文件，这个文件是用来生成aclocal.am文件和configure文件的，大家可以看到configure.in内容是一些宏定义，这些宏经autoconf处理后会变成检查系统特性、环境变量、软件必须的参数的shell脚本。：

![](manual/programming/makefile and automake/3.jpg)
```
dnl
这个宏后面的内容不会被处理，可以视为注释   
 
AC_INIT(FILE)
该宏用来检查源代码所在路径，autoscan 会自动产生，一般无须修改它。

AM_INIT_AUTOMAKE(PACKAGE,VERSION)  
这个是使用 Automake 所必备的宏，PACKAGE 是所要产生软件的名称，VERSION 是版本编号。    

AC_PROG_CC
检查系统可用的C编译器，若源代码是用C写的就需要这个宏。   
 
AC_OUTPUT(FILE)
设置 configure 所要产生的文件，若是Makefile ，configure 便会把它检查出来的结果填充到Makefile.in 文件后产生合适的 Makefile。  
  
实际上，在使用 Automake 时，还需要一些其他的宏，这些额外的宏我们用 aclocal来帮助产生。执行 aclocal会产生aclocal.m4 文件，如果没有特别的用途，不需要修改它，用 aclocal 所产生的宏会告诉 Automake如何动作,aclocal是一个perl 脚本程序。
```

4.aclocal.am文件的生成。同上面所说aclocal.m4文件是根据configure.in生成的，所以如果你这里aclocal.am文件没有成功生成，那么就是你configure.in文件有错误。

![](manual/programming/makefile and automake/4.jpg)

5.aclocal.am和configure.in都成功生成后，我们就可以用这两个文件来生成configure脚本了，也就是我们执行的./configure。

![](manual/programming/makefile and automake/5.jpg)

6.下面我们又要编写一个文件Makefile.am，它是用来和configure.in文件一起生成Makefile.in的，再调用./configure的时候，就将Makefile.in文件自动生成Makefile文件了。所以Makefile.am文件是比Makefile文件更高的抽象。关于Makefile.am的格式有很多，我这里只是抛砖引玉调用一些常见的用法，更多的类型需要到网上去查，如图：

![](manual/programming/makefile and automake/6.jpg)

7.输入automake生成Makefile.in文件（需要加参数--add-missing）。如图：

![](manual/programming/makefile and automake/7.jpg)

8.现在就可以直接打包成为一个源码包了，下面我们来实验这个源码包能不能使用。首先执行./configure生成Makefile文件：

![](manual/programming/makefile and automake/8.jpg)

9.有了Makefile文件，就可以来编译这个软件了。输入make，编译成功,并且生成了我们需要的可执行文件hello：

![](manual/programming/makefile and automake/9.jpg)

10.执行试试看,程序成功执行：

![](manual/programming/makefile and automake/10.jpg)

11.最后一步安装了，默认安装进/usr/local目录里，也是成功进行，可以直接输入命令了。

![](manual/programming/makefile and automake/11.jpg)