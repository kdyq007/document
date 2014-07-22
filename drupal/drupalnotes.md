1.超链接会调转到其它页面。想让它只在一个页面跳转模块

答：删除<a></a>中的target属性

```
<a target=\"_blank\" href=\"easdata?id={$one->id}\">查看详情</a>
改为：
<a href=\"easdata?id={$one->id}\">查看详情</a>
```


2.页面出现`http://10.0.128.220:8000/?q=easdata?id=6`这种请求，导致不能访问。

答：

a.需要启用`配置----简洁连接----启用`。

注：启用简洁连接需要开启apache或nginx的`rewrite`功能

apache:

LoadModule rewrite_module /usr/lib/apache2/modules/mod_rewrite.so

nginx:

```
server
  {
...
    location / {
        #启用rewrite
		if (!-e $request_filename) {
            rewrite ^([_0-9a-zA-Z-]+)?(/wp-.*) $2 last;
            rewrite ^([_0-9a-zA-Z-]+)?(/.*\.php)$ $2 last;
            rewrite ^ /index.php last;
        }
    }
...
  }
```

b.将href超链接修改为这样的格式：

"<a href=\"?q=easdata&id={$one->id}\">查看详情</a>"
