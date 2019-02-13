---
layout: post
title: "那一次，Python 让我彻底「沦陷」"
date: 2019-01-29 00:38:57
comments: true
reward: true
tags: 
	- Python
    - Restful API 
---

如果你问我为什么痴迷于 Python 的，那我就会从自己搭建简易的邮件报警服务说起，这件事情让我觉得 Python 实在是太高效了，学习的性价比非常高：作为一个 Python  小白，我能在两三天的时间内搭建一个稳健的邮件报警服务，至今仍在使用。从那以后，我学习 Python 劲头一发不可收拾，至今仍乐此不彼。人生苦短，我用 Python，这真是至理名言。

<!-- more -->

现在看来是挺简单的，也就是实现一个 url 接口，也就是 API，对此 api 提交 post 请求，后台就会按照 post 提交的数据进行邮件报警信息的发送。当时是挺有成就感的，因此现在想写出来分享下，其实 RESTful API 的开发套路都差不多，如果你感兴趣的话，请向下阅读。

2017 年的 6 月 28 日，由于工作上需要对很多作业的跑批进行监控，需要及时知道哪些作业报错，以便及时处理。偷懒的解决方法也就是找运维的同事按需求编写监控脚本，放在 zabbix 平台进行监控，走短信平台。由于当时系统还不是非常健壮，监控的需求会频繁变动，我决定先自己监控一段时间，稳定了再放 zabbix ，再者，走短信平台，每条短信还是有成本的，短信内容也有字数限制，而且不能发送更多日志信息。

于是我想到了邮件，邮件几乎是 0 成本，没有字数限制，而且内容可以有文字，图片，附件等，邮件客户端都会及时推送提醒，而且主流邮箱都有短信提醒功能，这就可以确保及时收到。基于以上原因，我决定自己尝试写个邮件报警程序。当时完全是个 Python 小白，从来没有用过 Python 写过任何程序。报着试试看的心态，随意地搜索了下「Python 邮件」定位到了 [Python3 SMTP发送邮件](http://www.runoob.com/python3/python3-smtp.html)，自己尝试了下，竟然一下就，成功了，看来 Python 发邮件还是相当简单的。

```python
#!/usr/bin/python3
 
import smtplib
from email.mime.text import MIMEText
from email.header import Header
 
# 第三方 SMTP 服务
mail_host="smtp.XXX.com"  #设置服务器
mail_user="XXXX"    #用户名
mail_pass="XXXXXX"   #口令 
 
 
sender = 'from@runoob.com'
receivers = ['429240967@qq.com']  # 接收邮件，可设置为你的QQ邮箱或者其他邮箱
 
message = MIMEText('Python 邮件发送测试...', 'plain', 'utf-8')
message['From'] = Header("菜鸟教程", 'utf-8')
message['To'] =  Header("测试", 'utf-8')
 
subject = 'Python SMTP 邮件测试'
message['Subject'] = Header(subject, 'utf-8')
 
 
try:
    smtpObj = smtplib.SMTP() 
    smtpObj.connect(mail_host, 25)    # 25 为 SMTP 端口号
    smtpObj.login(mail_user,mail_pass)
    smtpObj.sendmail(sender, receivers, message.as_string())
    print ("邮件发送成功")
except smtplib.SMTPException:
    print ("Error: 无法发送邮件")
```

看到这里我离目标已经近了一步，我可以将上面的代码改写，并封装成一个 Python 类，提供 send_mail(receivers, messages) 函数供报警程序调用就可以了，这样就解决了所有 Python 程序的报警问题。

问题是，如果非 Python 程序呢，我也想到了简单的解决方法，就是编写一个 Shell 脚本来调用 Python 程序，通过参数传递的方式来达到发邮件的目的，其他非 Python 程序只要调用这个 Shell 程序即可。

仔细一想，仍不是很完善，如果其他机器的程序想要调用这个脚本呢，那就需要把 Shell 脚本复制过去，这显然是麻烦的，后序如果 程序要更新，全都得再来一遍，而且会暴露邮件的密码。

幸好我知道有个东西叫 [RESTful API](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)，如果能实现发送邮件这个 API 的话，无论什么程序，在哪个终端，只要能访问该 API 都可以便捷的发邮件，而且只需要在服务端部署一次，就可以达到处处可用的效果，一劳永逸，后续即使有更新也是非常方便的。

接着我搜索 「Python RESTfulAPI」, 我了解到了 Django、Django REST FrameWork 等框架可以轻松实现，我便开始熟悉 Django 与 Django REST FrameWork，发现 Django 框架已经集成了邮件功能，只用 Django 就可以实现以上 RESTfulAPI。

## 手摸手，一步一步实现邮件报警 API

一步一步来，做起来，发现很简单。

#### 1、新建一个 Django 项目。
对 Django 还不熟悉但感兴趣的同学有时间可以去官网先逛逛，[https://docs.djangoproject.com/en/2.1/intro/tutorial01/](https://docs.djangoproject.com/en/2.1/intro/tutorial01/) 跟着教程一步一步来，不到 1 小时，你就可以搭建一个简单的网站。

**先用 pip 安装 django**
```python
pip install django
```
上述方法是在线安装 django，如果环境是离线的，请先用 pip 下载 django ，再将文件拷贝到离线环境使用 pip 安装，如下所示：
```python
mkdir django
cd django
pip download django
将 django 目录拷贝到离线环境，并进入  django 目录执行即可
pip install django --no-index --find-links ./
```

**新建api项目**

```python
$ django-admin startproject api
```
此时会生成 api 目录，内部有一个 manage.py 和 api 目录。

**在 api 项目下新建 mailapi 应用**
```python
$ cd api
$ ls
api  manage.py
$ python manage.py startapp mailapi
$ ls
api  mailapi  manage.py
```

**修改 Django 配置文件，使用本地时区，允许非本地 IP 访问**

修改 api/setting.spy

```python
ALLOWED_HOSTS = ['*']  #允许其他 IP 访问此网站
TIME_ZONE = 'Asia/Shanghai' #使用本地时区
```

**启动测试，确定无错误**

```python
$ python manage.py makemigrations
No changes detected
$ python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying sessions.0001_initial... OK
$ python manage.py runserver 0.0.0.0:8001
Performing system checks...

System check identified no issues (0 silenced).
January 28, 2019 - 00:23:50
Django version 2.0.7, using settings 'api.settings'
Starting development server at http://0.0.0.0:8001/
Quit the server with CONTROL-C.
```
我这里使用了 8001 端口，使用一个不冲突的端口就可以，如果不指定，则默认为 8000 。

在浏览器打开 http:ip:8001 出现以下页面说明项目已成功启动，可以进行 api 开发了，也可以在其他机器上访问，这里的 ip 就是项目所在机器的 ip 地址。
![](https://upload-images.jianshu.io/upload_images/12989993-e163295e42a60e01.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



#### 2、开发 mailapi


**修改配置文件**
django 发邮件使用 settings.py 中配置的 smtp 邮件服务器，端口、用户名、密码等信息，因此在开始前先修改 api/settings.py 添加以下信息

```python
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.163.com' 
EMAIL_PORT = 25
EMAIL_HOST_USER = 'xxxxx@163.com'
EMAIL_HOST_PASSWORD = '*****'
EMAIL_USE_LOCALTIME = True
```

并在 api/settings.py 中的 INSTALLED_APPS  添加 mailapi 应用 


```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'mailapi.apps.MailapiConfig',
]
```

**修改路由文件 urls.py**

修改 api/urls.py，内容如下所示：

```python
from django.contrib import admin
from django.urls import path,include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('mailapi.urls')),
]
```
新增一个文件： mailapi/urls.py，就是上步中 include 的目标文件，内容如下

```python
from django.urls import path

app_name = 'mailapi'

from . import views

urlpatterns = [
    path('sendemail/', views.send_email, name='sendemail'),
]
```


**在视图文件中加入发邮件的视图函数**

修改  mailapi/views.py ，加入以下内容：

```python
from django.http import JsonResponse
# Create your views here.
from django.core.mail import send_mail
from django.views.decorators.csrf import csrf_exempt

#先禁用防跨站请求伪造功能，方便 curl post 测试和调用
@csrf_exempt
def send_email(request):
    return_data = {'code':0,'message':'send email successful.'}
    
    subject = request.POST.get('subject', '')
    message = request.POST.get('message', '')
    from_email = request.POST.get('from_email', '')
    
    #设置 from_email 的默认值
    if from_email == '' or from_email is None:
    	from_email = 'zhengzheng@wjrcb.com'
    to_email = request.POST.get('to_email', '')
    

    print("subject",subject)
    print("message",message)
    print("from_email",from_email)
    
    if subject and message and to_email:
        try:
            to_email = to_email.split(';') #多个收件人以;分隔
            print("to_email",to_email)
            send_mail(subject, message, from_email, to_email)
        except BadHeaderError:
            return_data['code'] = 1
            return_data['message'] = 'Invalid header found.'
    else:
        # In reality we'd use a form class
        # to get proper validation errors.
        return_data['code'] = 2
        return_data['message'] = 'Make sure all fields are entered and valid.'
    
    return JsonResponse(return_data)
```

#### 3、发送邮件测试

先启动 django 项目：
```python
$ python manage.py runserver 0.0.0.0:8001
Performing system checks...

System check identified no issues (0 silenced).
January 28, 2019 - 11:57:56
Django version 2.0.7, using settings 'api.settings'
Starting development server at http://0.0.0.0:8001/
Quit the server with CONTROL-C.

```
再开启一个新的终端/命令窗口，使用 curl 工具来提交 post 请求，其中 from_email 可不写，默认值见视图函数。也可以不使用 curl 工具，只要能对 url 发送 post 请求即可。

如果是 curl ，则输入以下内容进行测试：

```python
$ curl -d "subject=邮件报警测试&message=这是一个邮件测试&to_email=somenzz@163.com" "http://localhost:8001/api/sendemail/"
{"code": 0, "message": "send email successful."}
```
注意，大部分系统的命令窗口默认是 UTF-8 编码，但 Windows 除外，如果在 Windows 系统下执行 curl（在 git bash 窗口中可以使用 curl）， 为了防止出现乱码，需要在前面指定字符集编码，如下所示：

```python
$ curl -H "content-type: application/x-www-form-urlencoded; charset=gbk" -d "subject=邮件报警测试&message=这是一个邮件测试&to_email=somenzz@163.com" "http://localhost:8001/api/sendemail/"
```

我们看到返回了 send email successful ，符合预期，然后再看下服务端的返回的信息：

```python
subject 邮件报警测试
message 这是一个邮件测试
from_email somenzz@163.com
to_email ['somenzz@163.com']
[28/Jan/2019 11:58:25] "POST /api/sendemail/ HTTP/1.1" 200 48
```
可以看出的确打印出了相应的信息，并返回了 200，说明发送成功，再检查一下邮件，确实收到了，大功告成。

![image.png](https://upload-images.jianshu.io/upload_images/12989993-8cfae7476e3cac96.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


实际使用中可以在后台运行 python manage.py runserver，防止因窗口关闭导致进程退出，如下所示：

```python
$ nohup python manage.py runserver 0.0.0.0:8001 &
```



#### 4、加入日志功能
创建 log 目录，存放日志信息。

```python
$ mkdir log && ls
api  db.sqlite3  log  mailapi  manage.py
```
日志都会遵循一定的格式，比如日间格式，日志级别，行号等，也需要指定日志输出位置，是文件还是终端的屏幕等。

此时修改 api/settings.py ，在文件末尾添加以下内容：

```python
#日志功能设置

LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {process:d} {thread:d} {message}',
            'style': '{',
        },
        'simple': {
            'format': '{levelname} {message}',
            'style': '{',
        },
    },


    'handlers': {
        'file': {
            'level': 'INFO',
            'class': 'logging.FileHandler',
            'filename': os.path.join(BASE_DIR,'log/info.log'),
            'formatter': 'verbose',
        },
        'console': {
            'level': 'INFO',
            'class': 'logging.StreamHandler',
            'formatter': 'verbose',
        },
    },

    'loggers': {
        'django': {
            'handlers': ['file','console'],
            'level': 'INFO',
            'propagate': True,
        },
    },
}

```
这些设置对于熟悉 python 标准库 logging 模块的同学已经非常熟悉了，如果暂时不熟悉也没关系，后面可以慢慢研究，先加上再说。

**修改视图函数 send_email，增加 ip 的获取功能，同时记录邮件相关的信息**，如下所示：

```python
from django.shortcuts import render
from django.http import JsonResponse
# Create your views here.
from django.core.mail import send_mail
from django.views.decorators.csrf import csrf_exempt


##记录日志
import logging
logger=logging.getLogger('django')


#先禁用防跨站请求伪造功能，方便 curl post 测试和调用
@csrf_exempt
def send_email(request):

    #获取请求方的 ip 地址
    ip=""
    if 'HTTP_X_FORWARDED_FOR' in  request.META:
        ip =  request.META['HTTP_X_FORWARDED_FOR']  
    else:  
        ip = request.META['REMOTE_ADDR']


    return_data = {'code':0,'message':'send email successful.'}
    
    subject = request.POST.get('subject', '')
    message = request.POST.get('message', '')
    from_email = request.POST.get('from_email', '')
    if from_email == '' or from_email is None:
    	from_email = 'somenzz@163.com'
    to_email = request.POST.get('to_email', '')
    

    print("subject",subject)
    print("message",message)
    print("from_email",from_email)
    
    if subject and message and to_email:
        try:
            to_email = to_email.split(';') #多个收件人以;分隔
            print("to_email",to_email)
            send_mail(subject, message, from_email, to_email)
        except BadHeaderError:
            return_data['code'] = 1
            return_data['message'] = 'Invalid header found.'
    else:
        # In reality we'd use a form class
        # to get proper validation errors.
        return_data['code'] = 2
        return_data['message'] = 'Make sure all fields are entered and valid.'
    ##记录日志
    logger.info(f"ip = {ip}, subject = {subject },message = {message }, from_email = {from_email }, to_email = {to_email }, return_data = {return_data }")
    return JsonResponse(return_data)

```
这里关键点是以下两点：一是获取请求端的 ip 地址，二是日志记录。

获取请求方的 ip 地址
```python
    #获取请求方的 ip 地址
    ip=""
    if 'HTTP_X_FORWARDED_FOR' in  request.META:
        ip =  request.META['HTTP_X_FORWARDED_FOR']  
    else:  
        ip = request.META['REMOTE_ADDR']
```
记录日志代码如下：
```python
 logger.info(f"ip = {ip}, subject = {subject },message = {message }, from_email = {from_email }, to_email = {to_email }, return_data = {return_data }")
```
根据开始时 handlers 的设置，同时打印到屏幕并输出到日志文件 info.log。再次启动 django 项目，并发送邮件测试，发现日志已经记录在 log/info.log 中，内容如下：

```python
INFO 2019-01-28 12:19:37,068 views 26852 140248897447680 ip = 127.0.0.1, subject = 邮件报警测试,message = 这是一个邮件测试, from_email = somenzz@163.com, to_email = ['somenzz@163.com'], return_data = {'code': 0, 'message': 'send email successful.'}
INFO 2019-01-28 12:19:37,068 basehttp 26852 140248897447680 "POST /api/sendemail/ HTTP/1.1" 200 48
```

同时在终端界面也有相应的信息打印：

```python
System check identified no issues (0 silenced).
January 28, 2019 - 12:19:32
Django version 2.0.7, using settings 'api.settings'
Starting development server at http://0.0.0.0:8001/
Quit the server with CONTROL-C.
subject 邮件报警测试
message 这是一个邮件测试
from_email somenzz@163.com
to_email ['somenzz@163.com']
INFO 2019-01-28 12:19:37,068 views 26852 140248897447680 ip = 127.0.0.1, subject = 邮件报警测试,message = 这是一个邮件测试, from_email = somenzz@163.com, to_email = ['somenzz@163.com'], return_data = {'code': 0, 'message': 'send email successful.'}
INFO 2019-01-28 12:19:37,068 basehttp 26852 140248897447680 "POST /api/sendemail/ HTTP/1.1" 200 48
```

日志功能就此告一段落。


#### 5、uwsgi 生产环境部署


1、安装 uwsgi

uwsgi:https://pypi.python.org/pypi/uWSGI

uwsgi 参数详解：http://uwsgi-docs.readthedocs.org/en/latest/Options.html

```python
pip install uwsgi
uwsgi --version    # 查看 uwsgi 版本
```

在项目 api 目录下新建 uwsgi 配置文件 uwsgi_api.ini, 加入以下内容：

```python
[uwsgi]
#socket = :8001
http = :8001
master = true
chdir = /home/aaron/web/api
wsgi-file = api/wsgi.py
processes = 4
threads = 10
vacuum = true         //退出、重启时清理文件
max-requests = 1000   
virtualenv = /home/aaron/pyenv
pidfile = /home/aaron/web/api/uwsgi_api.pid
#使进程在后台运行，并将日志打到指定的日志文件或者udp服务器
daemonize = /home/aaron/web/api/log/uwsgi.log
```
在命令中执行

```python
$ uwsgi uwsgi_api.ini
```
这样就使用了 uwsgi 服务器来驱动 django 项目，而不是那个有点 django 自带的较弱的服务器。上面的配置确保服务会自动转后台运行。日志文件会自动记录在 /home/aaron/web/api/log/uwsgi.log 中，此时会产生两个日志文件，info.log，uwsgi.log。还记得日志配置那里是这样配置的：
```python
    'loggers': {
        'django': {
            'handlers': ['file','console'],
            'level': 'INFO',
            'propagate': True,
        },
    },
```
其中 info.log 就是 handlers 中的 file 产生的，uwsgi.log 的内容是由 uwsgi 本身的日志 + handlers 中的 console 产生的日志。

**如果要关闭此服务**，只需要 kill 掉后台进程即可，命令如下：

```python
$ ps -ef|grep "uwsgi_api" 
$ ps -ef|grep "uwsgi_api" |grep -v grep | awk '{print $2}' |xargs kill -9
```
也可以看下 /home/aaron/web/api/uwsgi_api.pid 中的进程号，杀掉此进程即可。

到此已经可以结束了，如果想**使用 nginx 再做一层代理**，需要先修改 uwsgi 的配置文件，
使用 socket, 如下所示：

```python
[uwsgi]
socket = :8001
#http = :8001
master = true
chdir = /home/aaron/web/api
wsgi-file = api/wsgi.py
processes = 4
threads = 10
vacuum = true         //退出、重启时清理文件
max-requests = 1000   
virtualenv = /home/aaron/pyenv
pidfile = /home/aaron/web/api/uwsgi_api.pid
#使进程在后台运行，并将日志打到指定的日志文件或者udp服务器
daemonize = /home/aaron/web/api/log/uwsgi.log
```

再修改 nginx 配置文件：
```python
server {
        listen       80;
        server_name  localhost;
        
        location / {            
            include  uwsgi_params;
            uwsgi_pass  127.0.0.1:8001;              //必须和 uwsgi 中的设置一致
            uwsgi_param UWSGI_SCRIPT api.wsgi;  //入口文件，即 wsgi.py 相对于项目根目录的位置，“.”相当于一层目录
            uwsgi_param UWSGI_CHDIR /home/web/api/;       //项目根目录
            index  index.html index.htm;
            client_max_body_size 35m;
        }
    }
```
最后启动 nginx 
```python
$ /usr/local/nginx/sbin/nginx
```
此时访问 http://ip/api/sendemail/ 会自动请求 http://ip:8001/api/sendemail/，但如果在浏览器中访问，地址栏会显示： http://ip/api/sendemail/ ，不体现端口号是因为 80 端口是 http 的默认端口。


#### 6、报警功能的使用

如果是 shell 程序的话，直接使用 curl 对 "http://localhost:8001/api/sendemail/" 发送 post 请求即可。如果是其他主机请将 localhost 改为 API 服务所在的机器的 IP 地址。

对 subject，message，from_email, to_email 参数化，to_email 可以是一个收件地址，可以是多个，如果是多个请用 ; 分隔。

```python
curl -d "subject=邮件报警测试&message=这是一个邮件测试&to_email=somenzz@163.com;othermail@xx.com" "http://localhost:8001/api/sendemail/"
```
根据实际情况加入监控脚本中即可。

其他编程语言，都有 http 或 url 的库，直接调用，对 mailapi 做 post 请求即可。

**优化**

如果实际使用中短时间会有大量的邮件发送，官方推荐使用 send_mass_mail() ,函数原型如下：

```python
send_mass_mail(datatuple, fail_silently=False, auth_user=None, auth_password=None, connection=None)
```
django 的 send_mass_mail 与 send_mail 的区别就是 send_mail 每执行一次就连接一次邮件服务器，而  send_mass_mail 发送消息时只连接一次，因此  send_mass_mail 稍微更高效些。详细情况可以了解 Django 官网介绍：

https://docs.djangoproject.com/en/2.1/topics/email/

在公众号 somenzz 后台回复「mail 」获取项目源码，欢迎与我交流。

（完）

如果你觉得文章对你有帮助，请给个赏吧，一分钱也是爱。

![个人公众号](/assets/img/wechat2.jpg)
