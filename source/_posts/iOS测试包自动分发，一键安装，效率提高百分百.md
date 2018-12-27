---
title: iOS测试包自动分发，一键安装，效率提高百分百
date: 2016-06-22 13:22:41
tags: iOS测试包
---
一个可以让你快速、方便、一次配置，终生受益的测试包分发教程。你还在傻傻的用airdrop,qq么？

#使用环境：

适合iOS开发者，常需要发布测试包给各类人员，那么以后再也无需多余操作，一键搞定。公司有内网服务器，或用Mac os的同学都可以使用。非越狱手机可以使用，只要正常绑定过证书就没有问题。

以下是教程，相当简单。
服务器ip以192.168.1.188为例，端口8080

#第一步，配置run script打包ipa并完成ipa上传

部署过程，Xcode中打开target->build phases->add build phase->add run script如图添加如下代码，并根据自己使用环境做一下调整。

```
# shell script goes here

# compress application.
if [ "${CONFIGURATION}" = "ad_hoc" ]; then #判断发布版本

/bin/mkdir $CONFIGURATION_BUILD_DIR/Payload

/bin/cp -R $CONFIGURATION_BUILD_DIR/InstaSoccer.app $CONFIGURATION_BUILD_DIR/Payload

/bin/cp isoccer/icon/iTunesArtwork $CONFIGURATION_BUILD_DIR/iTunesArtwork

cd $CONFIGURATION_BUILD_DIR

# zip up the Instasoccer directory

/usr/bin/zip -r InstaSoccer.ipa Payload iTunesArtwork

/usr/bin/scp InstaSoccer.ipa sshuser@192.168.1.188:~/ipa_publish/ #scp到服务器路径，如果用Mac本机开启服务器，可以用cp到webserver路径
fi
exit 0
```


#第二步，部署服务器。
可以用Mac os的Web共享，也可以自己用python开一个，当然也可以用内网服务器、外网服务器，要求极低，扔几个静态文件就可以。
关于Mac os 10.8在偏好设置里面已经没有了Web共享，要开启的话需要手动写一下配置文件，方法请自搜。
 [点这里下载样例](http://www.minroad.com/wp-content/uploads/2013/06/%E5%BD%92%E6%A1%A32.zip)
样例中有2个文件，index.html和Info.plist
index.html修改一处，“http://192.168.1.188:8080/Info.plist” 改为你相应的路径

```
<html>
<head>
<meta charset="utf-8" />
<title>Minroad一键安装</title>
</head>
<a style="font-size: 5em;" href="itms-services://?action=download-manifest&url=http://192.168.1.188:8080/Info.plist">install</a>
<html>

```
Info.plist,修改ipa路径（如果你用scp的话请查看你scp后的路径是否与之相同），icon，版本号，bundle id，程序名

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>items</key>
	<array>
		<dict>
			<key>assets</key>
			<array>
				<dict>
					<key>kind</key>
					<string>software-package</string>
					<key>url</key>
					<string>http://192.168.1.188:8080/InstaSoccer.ipa</string>
				</dict>
				<dict>
					<key>kind</key>
					<string>display-image</string>
					<key>needs-shine</key>
					<true/>
					<key>url</key>
					<string>http://192.168.1.188:8080/Icon.png</string>
				</dict>
				<dict>
					<key>kind</key>
					<string>full-size-image</string>
					<key>needs-shine</key>
					<true/>
					<key>url</key>
					<string>http://192.168.1.188:8080/Icon.png</string>
				</dict>
			</array>
			<key>metadata</key>
			<dict>
				<key>bundle-identifier</key>
				<string>com.minroad.appid</string>
				<key>bundle-version</key>
				<string>2.8.2</string>
				<key>kind</key>
				<string>software</string>
				<key>subtitle</key>
				<string>一键安装副标题</string>
				<key>title</key>
				<string>一键安装程序名</string>
			</dict>
		</dict>
	</array>
</dict>
</plist>
```

然后在启动webserver, 方法多了去了，提供一个python的，Mac os也可以用

cd 到当前目录
nohup python -m SimpleHTTPServer 8080 > /dev/null 2>&1 &

点击安装就自动安装了。省心省力！只要将网址收藏，以后分发的事与开发人员就无关咯

文本为转帖 出处在这里 ：http://www.minroad.com/?p=688


