# 如何抠下微信小游戏？

### 一、首先列出些我们需要的信息

微信小游戏主页：
[http://game.wechat.123u.com/html](http://game.wechat.123u.com/html)

公众号关注页：
[http://mp.weixin.qq.com/s?__biz=MzA5MTkzMTUzMw==&mid=200836369&idx=1&sn=6f99ed2f4e8a44a9fa8b9df9f9d19bd5#rd](http://mp.weixin.qq.com/s?__biz=MzA5MTkzMTUzMw==&mid=200836369&idx=1&sn=6f99ed2f4e8a44a9fa8b9df9f9d19bd5#rd)

游戏测试服务器（改好的游戏传到测试服务器）：
`10.0.128.219:/usr/share/nginx/html`

二维码生成页（根据游戏网址生成二维码，手机微信测试）：
[http://cli.im](http://cli.im)

我们的统计代码：
```
<script>
    var _hmt = _hmt || [];
    (function() {
        var hm = document.createElement("script");
        hm.src = "//hm.baidu.com/hm.js?74100d7613bf120f0dc79d50b7661657";
        var s = document.getElementsByTagName("script")[0];
        s.parentNode.insertBefore(hm, s);
    })();
</script>
```

可能需要P图修改部分标志！！！

### 二、下面是一些常用的微信游戏页面，我们可以从这些上面找游戏，也可以百度。。

[http://g.9g.com/](http://g.9g.com)

[http://www.7k7k.com/m-android/play](http://www.7k7k.com/m-android/play)

==注意：找到游戏前先用微信测试以下，看手机是否能玩，否则改了半天手机是那个不能玩那就白忙乎了，像这个游戏原本到手机上就不能控制！==

### 三、下面我们来以一款叫做最强眼力的游戏来做例子

#### 1.首先我们要下载安装几个软件

- WebStorm,一款非常好用的JavaScript开发工具，共享下载地址：`\\fileserver\public\软件\前端开发\JetBrains.WebStorm`

- 一款浏览器，最好是chrome，我这里以它来操作

#### 2.对浏览器进行设置

打开最强眼力的游戏地址[http://www.wxmeimei.com/game/9/](http://www.wxmeimei.com/game/9/)，按下F12打开开发者工具，按下图的步骤分别点开如下按钮，最后点击Emulate开启模仿模式，这个功能能模拟手机来执行，让环境更真实。

![](manual/programming/wechatgame/1.jpg)


点击齿轮样子的setting，关闭缓存模式，方便测试游戏。

![](manual/programming/wechatgame/2.jpg)


#### 3.对webstorm进行设置

设置一个目录为专门存放游戏的目录，点击webstorm的File----Settings----Directories----Add Content Root,添加该目录为游戏根目录。

![](manual/programming/wechatgame/3.jpg)


如果js代码比较乱，可以让webstorm自动调整，点击Code----Reformat Code,它能自动调整该目录所有JS文件代码。

#### 4.下载该游戏和缺漏的游戏资源

在游戏页面按下ctrl+s，会自动弹出保存对话框，在游戏存放目录新建一个游戏目录，叫zqyl，打开，文件名保存为‘index.html’，保存类型选择为‘网页,全部’，点击保存。

![](manual/programming/wechatgame/4.jpg)


在webstrom中打开该目录index.html文件，在主页文件上右键----Debug 'index.html'，会自动打开默认的浏览器来调试，webstrom默认的浏览器在Run----Edit Configurations中设置。

![](manual/programming/wechatgame/5.jpg)


按出F12开发者工具，点击Console，这里会将已有的错误和找不到的文件都列出来，如果缺少什么，就将其下载下来，当然在webstrom下面的debug信息里也会有，有些图片在游戏结束时才会调用，所以，你可以将游戏快速跪掉，查看缺漏文件。

![](manual/programming/wechatgame/6.jpg)


如果有文件not found，那么重新打开一个页面，跳到原始游戏地址，打开开发者工具，点击Resoures，在左边的Frames里会有调用的文件信息，images、js、css等，比如我现在需要下载2000.png这个图片，点击右下方的URL在新页面打开，然后直接拖拽进目录即可。

![](manual/programming/wechatgame/7.jpg)


#### 5.文件格式化分类

为了方便日后的游戏管理与代码阅读，所以有必要对游戏内容进行归类。在游戏根目录新建几个子目录，src、res、res/images、res/css,如果有声音文件就有必要再建一个res/sounds,src目录存放js源码，res就是存放各种资源文件。将原本index_file目录里的文件复制至相应的新建目录里，就可以将index_file目录删除了，最好是重命名以作备份。

![](manual/programming/wechatgame/8.jpg)


#### 6.开始修改游戏代码

- 首先将我们复制的文件路径改正过来，如pyq.png，在index.html和各个js文件里搜索，如图可以看到原本的路径是`./index_files/pyq.png`，我们将其修改为`./res/images/pyq.png`,左边还有一个`./index_files/9.js`，将其对应的改为`./src/9.js`,这一步可以和上步同时来做，复制一个资源就改一下路径，以防遗漏。

![](manual/programming/wechatgame/9.jpg)


- 其次就是修改一些常见的游戏连接和关键词，如这里，是一个跳至原始游戏官网的连接，可以将其改为我们自己的欢乐小游戏主页，当然，文字也可以改。

![](manual/programming/wechatgame/10.jpg)


- 下面来看看这样一段script代码，很明显这是一段5拉网的统计代码，我们可以将其完全删掉，替换成我们自己的统计代码。统计代码类型有很多，我们自己的就是百度的，差不多都是包含在`<script></script>`段中的，由此可见`17224329.js`和`icon_6.gif`都是统计代码调用，我们可以完全的将其删除，不会影响到游戏。

![](manual/programming/wechatgame/11.jpg)


- 如下面这段，一定要注意分享链接，我们要填入游戏主页地址，这样分享出去的游戏才能直接点开就玩；还有这个外部站点的图片，也可以存进我们目录里，链接一定要是绝对路径。

![](manual/programming/wechatgame/12.jpg)


- 调试游戏，发现不能正常运行，看webstorm调试信息里提示找不到9.css，原来9.css文件路径还没有改正过来，搜索9.css，将其路径修正为`./res/css/9.css`即可。

![](manual/programming/wechatgame/13.jpg)


![](manual/programming/wechatgame/14.jpg)


- 看看9.css中的一段代码原本链接是绝对路径，我们可以改为绝对路径，当然，如下我改的是相对路径，因为这个css经过我们移动后，是在目录`res/css/`下面，在这里调用图片，就要用`../images/`先返回到`res/`目录再进入到`images/`目录，这样才能正常调用图片。

![](manual/programming/wechatgame/18.jpg)

- 再说一个eval的混淆，如果在代码中看到有eval这个函数，或修改游戏后，仍然有一个其它位置的链接，那肯定是代码混淆了，如下图。这时可以将这段eval函数复制到[http://dean.edwards.name/unpacker](http://dean.edwards.name/unpacker/)这个网址上解压，就可以看到原始的代码了，轻松修改链接，然后用解压过的代码直接替换原始的混淆代码。

![](manual/programming/wechatgame/16.jpg)


- 最后将游戏代码过一遍，index文件和各个js文件，检查是否有需要修改的地方。

#### 7.测试游戏

现在再来调试游戏就可以正常玩了，电脑测试成功，然后就是手机上微信测试，将修改好的游戏上传至微信测试机上`10.0.128.219`，打开xshell，登录，跳转到`/usr/share/nginx/html`目录下，将游戏文件夹上传上去。

![](manual/programming/wechatgame/17.jpg)


然后就可以通过[http://10.0.128.219/zqyl/index.html](http://10.0.128.219/zqyl/index.html)来访问了，为了方便手机输入网址麻烦，可以将它做成二维码，微信一扫就可以玩咯！


最后还要注意检查是否所有的链接都修正为我们自己的、是否所有的字符广告也是我们自己的，多余的广告、统计都删除。