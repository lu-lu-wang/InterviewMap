#本地起服务
[本地起服务](https://www.jianshu.com/p/90d5fa728861)

[添加文件到服务器](https://www.jianshu.com/p/c3d183d095bc)

##用Apache服务
开启apache
> sudo apachectl start

重启apache
> sudo apachectl restart

关闭apache
> sudo apachectl stop

打开safari访问127.0.0.1
> it vorks!

####说明服务已生效，这个页面时默认页面，可以自行修改
apache的修改路径
> /Library/WebServer/Documents

将自己的网页和资源丢进去就可以打开了
###测试
1. 创建一个文件，如index.html
2. 浏览器访问127.0.0.1/index.html
3. IP可换成自己电脑的IP地址，同一局域网设备都可以访问服务器内容。


###！！！记得关闭服务器，否则会一直消耗电脑内存
