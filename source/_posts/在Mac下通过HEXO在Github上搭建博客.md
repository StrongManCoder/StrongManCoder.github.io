---
title: 在Mac下通过HEXO在Github上搭建博客
date: 2016-03-18 14:51:30
tags: [hexo,Mac, GitHub]
---
经过一番折腾，总算是把Hexo给弄好了。在这期间遇到了各种问题，网上有的教程也有点老了，这里就再写一篇。最新的教程可以去Hexo官网查看。

前期准备
安装Xcode
Hexo的编译可能依赖Xcode。这个直接从App Store上下载就好了，没什么难度。

安装node.js
Hexo是基于node.js的，所以要去官网上下载下来安装。版本可以选择稳定版(4.3.1)也可以选择最新版(5.7.0)。
需要注意的是，Hexo 3.1.1测试的最低版本为0.12，所以安装的版本不要太旧，之前看到网上装的0.8.4的版本，我也这么装，结果有一大堆的报错。

注册Github账户
在本地搭建好Hexo后可以将内容同步到github上，可以在网上浏览。
可以去Github官网上去注册，注册的过程我就不罗嗦了，具体的过程可以去这个页面上跳到Github的那部分去看。
其中配置SSH Keys的那部分，可以选择不配制，不配置的话以后每次提交的时候就需要手动输入账号密码，如果配置了的话就不需要了。

正式安装
因为安装包中有些内容在墙外，所以可以换淘宝源，或者用
`npm install -g hexo-cli –no-optional
`

来安装
然后进入你要安装的目录，如

`cd ~/Document/hexo`

然后安装

`hexo init
`
安装好之后不要忘记执行

`npm install
`
至此，就已经安装完毕了。是不是很简单呢？

后期部署
添加文章

`hexo new “postName”
`
其中postName是博客名。

生成静态页面

`hexo generate
`
或者也可以执行缩写

hexo g

本地启动
执行好上面的命令之后就可以在本地启用服务来看效果了。执行下面的命令：

`hexo sever
`
或缩写

`hexo s
`
看到 INFO Hexo is running at http://0.0.0.0:4000/. Press Ctrl+C to stop. 之后，就可以在浏览器中打开页面http://localhost:4000来看了。

上传至Github
安装git部署插件
在部署之前，首先我们要确认在你的Github帐号的Repository中有 用户名.github.io 的项目。
在确认之后，就可以执行命令

`npm install hexo-deployer-git –save
`
来安装插件

配置 _config.yml 文件
在Hexo安装的目录，如 ~/Document/hexo 中找到 _config.yml 文件。打开。
翻到最后，找到 deploy 字样，改成如下格式：


```deploy:
type: git
repo: https://github.com/用户名/用户名.github.io.git
branch: master
```


需要注意的是：冒号后面有一个空格；使用github可以不用写branch那一行。
如果要使用多个 deployer，可改成如下样式：

deploy:

type: git
repo:
type: heroku
repo:
同步
输入命令

hexo deploy

或者缩写

hexo d

来执行。
以后每次执行就可以依次输入下面三行命令：

hexo clean
hexo generate
hexo deploy

或者其缩写。


Hexo基本常用的命令就四个，而且还可以使用组合命令。基本命令如下：



```hexo g = hexo generate  #生成
hexo s = hexo server  #启动本地预览
hexo d = hexo deploy  #远程部署
hexo n "文章标题" = hexo new "文章标题"  #新建一篇博文
```
　　我通常是选用组合命令，操作更为效率。如果你使用搜狗输入法的话，可以自定义一个短语，比如我输入hs则出现hexo s -g命令。


```hexo s -g  #等同先输入hexo g，再输入hexo s
hexo d -g  #等同先输入hexo g，再输入hexo d
```




最后优化
插件
我使用了几个常见的插件：

从Wordpress迁移到Hexo

npm install hexo-migrator-wordpress –save

在 WordPress 仪表盘中导出数据(“工具(Tools)” → “发布(Export)” → “文章(WordPress)”)
插件安装完成后，执行下列命令来迁移所有文章。source 可以是 WordPress 导出的文件路径或网址。

hexo migrate wordpress

站点地图

npm install hexo-generator-sitemap –save

生成的sitemap.xml可以给搜索引擎收录使用。
如果要生成百度的sitemap，使用以下命令：

npm install hexo-generator-baidu-sitemap –save

RSS订阅

npm install hexo-generator-feed@1.0.3 –save

配置文件里经常看见的/atom.xml就是由这个插件生成的

