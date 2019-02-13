---
layout: post
title: "用 Python 打造在线家庭影院"
date: 2018-11-24 23:00:57
comments: true
reward: true
tags: 
	- nginx
	- django
---

我喜欢看电影，尤其是好的电影，我会看上三四遍，仔细感受电影带给我的另一种人生体验，不同时期，不同年龄段看相同的电影，体验也会不一样。比如你上学时期看周星驰的电影可能就仅看到了笑点，工作之年之后再看，也许你会觉得这蕴含着深刻的人生哲理。

以前下载的电影，放的到处都是，手机上，U 盘里，平板，台式机，笔记本上都保存有下载过的电影，而且有时候平板或手机空间不够，就不得不删除珍藏已久的电影，很是可惜。当要看电影时，一时却找不到自己曾下载过的电影，于是又在网上搜索，但是随着版权越来越被重视，看视频都要会员，或者付费观看（这一点是进步的，只有这样才会有更好的作品呈现。），没有会员就要忍受非常烦人的广告，而且未必是高清资源。想想曾经下载过的电影删除了，现在看可能要收费了，很是遗憾。

如果电影可以统一放在廉价的台式机硬盘上，再开启一个视频流服务器能让所有的联网设备直接在线播放就好了，这样就不用担心下载过的电影无法找到了，而且觉得好的电影可以随时推荐给家人和朋友观看。
<!-- more -->
我知道 Python 是可以干这个事情的，说干就干，当天晚上就做好了一个 demo。【如果你也有个想法想实现，那么请即刻行动起来，如果超过 72 个小时还没行动，你很可能再也不会去做了】

技术栈：python、django、nginx

感兴趣的和我一起动手做吧。以 windows 操作系统为例，其他系统可做参考。

## 1、下载并配置 nginx 

nginx 是什么？
>Nginx ("engine x") 是一款是由俄罗斯的程序设计师 Igor Sysoev 所开发高性能的 Web 和反向代理服务器，也是一个 IMAP/POP3/SMTP 代理服务器。在高连接并发的情况下，Nginx 是 Apache 服务器不错的替代品。

这里主要用 nginx 将 mp4 文件转化为流媒体，这样就可以直接在网页上播放 mp4 格式的电影，只需要简单的配置即可，不需要编写代码，非常简单。

登陆官网下载 nginx：[https://nginx.org/en/download.html](https://nginx.org/en/download.html)，下载如下图所示的稳定版本，解压即可使用。
![nginx-download.png](https://upload-images.jianshu.io/upload_images/12989993-6aa6b505545550b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

修改 nginx.conf ，在 http 部分添加如下所示的部分。
```sh
    server {
        listen       8080;
        server_name  localhost;
        location /movie {
            autoindex on;
            autoindex_exact_size off; 
            autoindex_localtime on;
            proxy_pass   http://127.0.0.1:8000/movie;
        }
        location ~ \.mp4{ 
              autoindex on;
              root  E:\\media;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
```
其中
```sh
        location ~ \.mp4{ 
              autoindex on;
              root  E:\\media;
        }
``` 
的作用是将 mp4 文件转化为流媒体的，只需要这一段，你就可以在浏览器上播放电影了，比如我在E:\media\ytza\[迅雷下载Www.99b.Cc]伊甸湖BD1024高清中英双字.mp4，我就可以在地址栏按下图所示的内容：
![image.png](https://upload-images.jianshu.io/upload_images/12989993-b6f75f14bbf38c90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

是不是有点小兴奋。

但是，你不可能记得每一个电影名称和路径，nginx 虽然也可能列出文件列表，但涉及到中文就会乱码，而且不太容易解决，这就需要简单的编程来解决文件路径显示的问题。接下来看 2。

## 2、使用 Django 显示本地电影列表

Django 是什么，相信你会想起电影《被解救的姜戈》，Django 就是读姜戈，第一个 D 不发音。
![image.png](https://upload-images.jianshu.io/upload_images/12989993-9996c1849d3f5c96.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>Python下有许多款不同的 Web 框架。Django 是重量级选手中最有代表性的一位。许多成功的网站和APP都基于 Django。Django 是一个开放源代码的 Web 应用框架，由 Python 写成。Django 遵守 BSD 版权，初次发布于 2005 年 7 月, 并于 2008 年 9 月发布了第一个正式版本 1.0 。Django 采用了 MVC 的软件设计模式，即模型 M，视图 V 和控制器 C。

Python 是什么就不用介绍了，下面直接展示如何使用 django 快速生成一个网站。
在命令窗口依次执行下面的命令：
```python
pip install django
django-admin startproject mysite
cd mysite
python manage.py startapp movie
```

1、编辑 movie 目录下的 views 文件如下所示：
```python
from django.shortcuts import render
import os
# Create your views here.


FILE_HOME_DIR = "e:/media/"
MEDIA = [ ".mp4",]


def movie_list(request):
    next = request.GET.get("next", '')
    print(f"next = {next}")
    path = "/".join(request.path.split("/")[2:])
    print(f"request.path= {request.path}")
    print(f"path = {path}")

    #print os.listdir(FILE_HOME_DIR+".none/")
    data = {"files":[], "dirs":[]}
    print(data)
    child_path = FILE_HOME_DIR+path+next
    print(f"child_path = {child_path}")
    data['cur_dir'] = path+next
    print(data)
    for dir in os.listdir(child_path):
        if os.path.isfile(child_path+"/"+dir):
            if os.path.splitext(dir)[1] in MEDIA:
                data['files'].append(dir)
        else:
            data['dirs'].append(dir)

    print(data)
    return render(request,"movie/index.html", data)
```
2、在 movie 目录下新建 templates/movie 文件夹，添到文件 index.html，内容如下所示：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>电影列表</title>
</head>
<body>
<ul>
{% for dir in dirs%}
    <li style="color: #462921;font-size: 20px;">
        {{ dir }} <a  href="/movie/{{ cur_dir|safe }}/?next={{ dir|safe }}">进入</a>
    </li>
{% endfor %}
{% for file in files%}
    <li style="margin-right: 20px;font-size: 20px;">
         {{ file }} <button><a href="/{{ cur_dir|safe }}/{{ file|safe }}" target="_blank">播放</a></button>
    </li>
{% endfor %}
</ul>

</body>
</html>
```
3、修改 mysite/urls.py 内容如下：
```python
from django.conf.urls import include,url
from django.contrib import admin

urlpatterns = [
    url(r'^polls/', include('polls.urls')),
    url(r'^movie/', include('movie.urls')),
    url(r'^admin/', admin.site.urls),
]
```
4、在movie 文件夹下添加 urls.py 内容如下：

```python
from django.conf.urls import include,url
from . import views
app_name = 'movie'
urlpatterns = [
    url(r'', views.movie_list, name="movie"),
]
```
5、修改 settings.py 下列内容的值
```python
INSTALLED_APPS = [
    'movie.apps.MovieConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
ALLOWED_HOSTS = ['*']
```
上述步骤对应的文件如下图所示：
![image.png](https://upload-images.jianshu.io/upload_images/12989993-4b6a35cbab9559d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
其他保持默认即可。
接着在 manage.py 所在的文件夹内，也就是 mysite 目录内执行

```python
python manage.py makemigrations
python manage.py migrate
python manage.py runserver 0.0.0.0:8000
```
即可启动会显示电影列表的网站，如下图所示：
![image.png](https://upload-images.jianshu.io/upload_images/12989993-a8f27e71ef552e89.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

此时 nginx 的这段代码就发挥作用了
```sh
        location /movie {
            autoindex on;
            autoindex_exact_size off; 
            autoindex_localtime on;
            proxy_pass   http://127.0.0.1:8000/movie;
        }
```
只要输入 127.0.0.1:8080/movie 就会自动转到 http://127.0.0.1:8000/movie 的服务上，这样你就看到了电影的目录，当点击对应的 mp4 文件时，nginx 自动转成流媒体为你播放。

在别的终端上看时，直接输入服务器所在的 IP 地址即可，如下图所示：

![](https://upload-images.jianshu.io/upload_images/12989993-00345478710b7902.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![进入目录](https://upload-images.jianshu.io/upload_images/12989993-f58ac31352b9957f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/12989993-18836505a7677053.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

有同学可能会问了，如果电影不是 mp4 格式的呢？ 由于 html5 仅支持直接播放 mp4 ，我想到的办法就是使用 ffmpeg.exe 将其他非 mp4 格式的电影转成 mp4，代码已经为你写好了，直接使用即可。
movie2mp4.py
```python
# -*- coding: utf-8 -*-
#!/usr/local/bin/python
# Time: 2017/8/30 22:17:47
# Description:
# File Name: movie2mp4.py

import os
import subprocess
import time
import logging


logger = logging.getLogger()
logger.setLevel(logging.INFO)
ch = logging.StreamHandler()
fh = logging.FileHandler(filename="./convert.log")
formatter = logging.Formatter(
    "%(asctime)s - %(name)s - line:%(lineno)d - %(levelname)s - %(message)s"
)
fh.setFormatter(formatter)
ch.setFormatter(formatter)
logger.addHandler(ch)  # 将日志输出至屏幕
logger.addHandler(fh)  # 将日志输出至文件

movie_path = r"e:\media"
movie_type = [".mkv", ".rmvb", ".avi",".flv",".MKV", ".RMVB", ".AVI",".FLV"]

for root, dir, files in os.walk(movie_path):
    for file in files:
        # print(file)
        shotname, extension = os.path.splitext(file)
        # print(shotname,extension)
        source_name = os.path.join(root, file)
        target_name = os.path.join(root, f"{shotname}.mp4")
        if extension in movie_type and not os.path.exists(target_name):
            logger.info(f"{source_name} -> {target_name}")
            bin_path = (
                r"D:\program\ffmpeg-20170830-2b9fd15-win64-static\bin\ffmpeg.exe"
            )
            #command = f"{bin_path} -i {source_name} -strict -2 {target_name}"
            command = f"{bin_path} -i {source_name} -c:v copy -c:a copy {target_name}"
            logger.info(command)
            try:
                a = subprocess.run(command, shell=True)
                if a.returncode == 0:
                    logger.info("convert successful.")
                    b = subprocess.run(f"del {source_name}", shell=True)
                    if b.returncode == 0:
                        logger.info("delete source successful.")
            except Exception as e:
                logger.info(f"conver error :{e}")

```
这样就可以愉快的享受自己搭建的家庭影院了。

##3、后续可优化的方向

1、加入电影分类，最好是自动分类。
2、加入权限控制，家里的小朋友只能看少儿宜的电影。
3、加入域名服务，可以在外网看家里的电影。
由于时间有限，后续如果有时间再弄吧。

如有需要获取源码，加微信公众号 somenzz ，后台回复 "家庭影院" 获取。

![somenzz.png](https://upload-images.jianshu.io/upload_images/12989993-ae5177517369027e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

