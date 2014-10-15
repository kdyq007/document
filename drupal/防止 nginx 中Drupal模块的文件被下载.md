防止 nginx 中Drupal模块的文件被下载

虽然 Drupal 是开源的，但自己开发的模块会牵涉到一些商业逻辑或业务机密。因此防止自己模块文件被下载就显得有必要了。如果你使用的Web服务器是 Apache，那么只要启用Apache的 rewrite 扩展就可以了，Drupal 自带的 .htaccess 文件已经做了处理。

如果你使用的Web服务器就Nginx，那么你可以在你的 Nginx 的配置文件中的 server 部分，添加如下的代码：

```
location ~* \.(engine|inc|info|install|make|module|profile|test|po)$
                        {
                          root /var/www/demo; //修改成你的网站路径
                          deny all;
                        }
```

然后重启你的Nginx服务器，就可以防止以 engine、inc、info、install、make、module等扩展名结尾的文件了。