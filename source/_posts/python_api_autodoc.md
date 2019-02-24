---
layout: post
title: "让 API 自动生成文档"
date: 2019-02-22 22:38:57
comments: true
reward: true
tags: 
	- Django 
	- rest framework
---

程序员最苦恼的事情莫过于写文档。由于业务口径频繁变更，因此很多接口也会频繁变更，频繁变更导致文档的维护是一件相当费时的事情，当优先级更高的事情袭来，更新文档反到成了次要工作，久而久之，文档就算有，也不是最新的，有些接口，干脆文档也不写了，口口相传了事。

<!-- more -->


没有文档，对于新手或者工作交接，是一件非常麻烦的事情，也不利于程序的传承。

那么，有没有这样一种程序，根据 api 函数的规范注释，及 api 的功能自动生成 api 的文档呢？这样一来，改接口，只要注释完善下，api 文档就自动生成，文档时刻保持最新，岂不省事。网上搜索了下，还真有大神实现了这样的框架。不得不感慨，没有程序员实现不了的好功能，只有程序员想不到的好方法。

实际上，一些流行的 web 框架已经原生集成了自动生成 api 文档的功能。比如我最近学习的  django rest framework 框架就可以自动生成 api 文档，有了这个功能，领导再也不用担心没有接口文档了。

下面对官方给和样例程序及自定义的 api 来自动生成文档，暂时不考虑 api 的权限及有选择的生成 api 文档功能，这些在深入学习之后，都不是难事。这些样例的作用在于快速展示如何自动生成 api 文档的功能，想深入了解的还是看下框架的源代码。

### 先开发 api

请先仿照  django rest framework 官方的教程快速实现一个 api。
[https://www.django-rest-framework.org/tutorial/quickstart/](https://www.django-rest-framework.org/tutorial/quickstart/)

比如，我这里的 api 视图代码就是这样子的：

1、官方的 api 
```python
##官方api

from django.contrib.auth.models import User, Group
from rest_framework import viewsets
from mail.serializers import UserSerializer, GroupSerializer


class UserViewSet(viewsets.ModelViewSet):
    """
    API endpoint that allows users to be viewed or edited.
    """
    queryset = User.objects.all().order_by('-date_joined')
    serializer_class = UserSerializer


class GroupViewSet(viewsets.ModelViewSet):
    """
    API endpoint that allows groups to be viewed or edited.
    """
    queryset = Group.objects.all()
    serializer_class = GroupSerializer

```

2、我自己写了一个发送邮件的 api ，代码如下：

```python
# Create your views here.
from rest_framework.views import APIView
from django.core.mail import send_mail, BadHeaderError
from rest_framework.response import Response
from .get_parameter import get_parameter_dic
import logging
logger = logging.getLogger('django')
##自定义api
class SendMail(APIView):
    """
    发送信息到指定人员邮箱
    """
    def post(self, request, format=None):
        """
        post:
        发送信息到指定人员邮箱
        参数列表：
            from_email： 发件人邮箱
            to_email: 收件人，多个收件人请使用英文逗号分隔隔开
            subject: 邮件主题
            message: 邮件正文
        """
        # 获取请求方的 ip 地址
        print(request.data)
        ip = ""
        if 'HTTP_X_FORWARDED_FOR' in request.META:
            ip = request.META['HTTP_X_FORWARDED_FOR']
        else:
            ip = request.META['REMOTE_ADDR']

        params = get_parameter_dic(request)
        subject = params['subject']
        message = params['message']
        from_email = params.get('from_email', 'somenzz@163.com')
        to_email = params['to_email']

        return_data = {'code': 0, 'message': f'send email to {to_email} successful.'}
        if subject and message and to_email:
            try:
                to_email = to_email.split(',')  # 多个收件人以;分隔
                print("to_email", to_email, type(to_email))
                send_mail(subject, message, from_email, to_email)
            except BadHeaderError:
                return_data['code'] = 1
                return_data['message'] = 'Invalid header found.'
        else:
            # In reality we'd use a form class
            # to get proper validation errors.
            return_data['code'] = 2
            return_data['message'] = 'Make sure all fields are entered and valid.'
        logger.info(
            f"ip = {ip}, subject = {subject },message = {message }, from_email = {from_email }, to_email = {to_email }, return_data = {return_data }")

        return Response(return_data)
```
这里使用了 post 方法，在 post 请求的 body 里可以传输 4 个参数，分别是 subject 、message、from_email、to_email。其中 from_email 有默认值，是 somenzz@163.com，因此这个参数也可以省略。

这里分享下 **django 框架获取参数的通用函数**。

django 框架获取参数有多种方式，如 get 请求中参数都会在 url 中传输，比如：http://xxx.com/api/?name=asdf&phone=13xxxx  这样。使用 request.query_params 中可以获取 name，phone 等参数，request.query_params 返回的数据类型为 QueryDict，QueryDict 转为普通 python 字典 query_params.dict()即可。

在 post 请求参数一般放在请求的 body 中, 但是仍可以放在 url 仍中，类似 get 的形式, 最终结果, 参数会有两部分组成, 一部分在 url 中, 一部分在http body 中, 但是非常不建议这样做。接下来的代码编写也不会考虑这样的情况, post 仅考虑所有参数都在 http body 中的情况。这样，无论是 post ，还是 get ，我们可以编写统一的 参数获取函数，如下所示：

```python
from django.http import QueryDict
from rest_framework.request import Request
def get_parameter_dic(request, *args, **kwargs):
    if isinstance(request, Request) == False:
        return {}

    query_params = request.query_params
    if isinstance(query_params, QueryDict):
        query_params = query_params.dict()
    result_data = request.data
    if isinstance(result_data, QueryDict):
        result_data = result_data.dict()

    if query_params != {}:
        return query_params
    else:
        return result_data
```
也是自定义 api 中 
```python
from .get_parameter import get_parameter_dic
```
使用的函数。

3、修改 settings.py 
在 INSTALLED_APPS  增加两项：
```python
INSTALLED_APPS = [
    'mail.apps.MailConfig', #自定义的 app
    'rest_framework', #导入 rest_framework
]
```
修改时区：

```python
TIME_ZONE = 'Asia/Shanghai'
```

增加发邮件的信息

```python
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.163.com'
EMAIL_PORT = 25
EMAIL_HOST_USER = 'somezz@163.com'
EMAIL_HOST_PASSWORD = '******'
EMAIL_USE_LOCALTIME = Tru
```

### 方法一、使用原生样式自动生成 api 文档 

修改项目总的 urls.py，加入文档 url 路由，如下所示：

```python
from django.contrib import admin
from django.urls import path,include
from rest_framework.documentation import include_docs_urls

from rest_framework import routers
from mail import views

router = routers.DefaultRouter()
router.register(r'users', views.UserViewSet)
router.register(r'groups', views.GroupViewSet)

# Create our schema's view w/ the get_schema_view() helper method. Pass in the proper Renderers for swagger

urlpatterns = [
    path('admin/', admin.site.urls),
    path('mail/',include('mail.urls')),
    path('',include(router.urls)),
    path('api-docs/', include_docs_urls(title="api接口文档")),

]

```
与原本的 urls.py 相比，其实就多了两行代码，
```python
from rest_framework.documentation import include_docs_urls
path('api-docs/', include_docs_urls(title="api接口文档")),
```
就是这两行代码，自动生成了 api 的文档。下面来看一下效果

![api1.png](https://upload-images.jianshu.io/upload_images/12989993-83099e20ae93de86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 我们可以看到这个 api 接口文档已经相当丰富了，左侧是所有的 api 列表，点击可以定位到相应的说明，也可以与点击 Source Code 查看多种语言调用 api 的样例代码。也可以点击 interact 按钮与 api 进行交互来测试  api，如下图所示：

![api2.png](https://upload-images.jianshu.io/upload_images/12989993-f20ae22bf03a505a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![api3.png](https://upload-images.jianshu.io/upload_images/12989993-54528c5950a1324e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![api33.png](https://upload-images.jianshu.io/upload_images/12989993-a8955d1c11fa9fad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

也可以在侧查看返回信息，及原始字符串 raw。这些 api 有个共同点就是使用 django rest framework 封装好的类来实现的，屏蔽了很多细节，现在我们看一下自定义的发邮件 api，看看它的交互如何？
![自定义的api](https://upload-images.jianshu.io/upload_images/12989993-90631d48021085f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到它获取到了 api 中的注释字符串。
![自定义的api 未发现参数框](https://upload-images.jianshu.io/upload_images/12989993-80dd9d903fd7cecc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们发现自定义的 api 没有对应的参数可以填写，这真让人郁闷。仔细啃官方的英文文档，终于在第二天实现了，方法如下：
修改自定义的 api 视图类，加入以下代码：

```python
    schema = AutoSchema(manual_fields= [
        coreapi.Field(name="subject", required=True, location="query", description="邮件主题"),
        coreapi.Field(name="message", required=True, location="query", description="邮件正文"),
        coreapi.Field(name="from_email", required=False, location="query", description="发件人"),
        coreapi.Field(name="to_email", required=True, location="query", description="收件人，多个使用逗号分隔"),
    ])

```
前提要导入以下包：
```python
from rest_framework.schemas import AutoSchema
import coreapi
```
再次查看自定义的 api，发现有变化了：
![自定义的api](https://upload-images.jianshu.io/upload_images/12989993-20c17ecc1c7025c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们发现，有了参数，但是描述信息不知道为什么没有获取到，如果有大神知道，请赐教。

下面交互，

![自定义的api交互成功](https://upload-images.jianshu.io/upload_images/12989993-92e0edce1e3f4a91.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

终于大功告成，后面有人来问 api 的事情，不用理他，向他扔这样一个 api 文档就可以了。

注意，这里依赖 coreapi ，使用过程中使用 pip 安装下即可

```python
pip install coreapi 
```

### 方法二、使用第三方库自动生成 api 文档 
这里介绍下 django-rest-swagger，使用方法如下：
1、先安装：
```python
pip install django-rest-swagger
```
2、加入到 INSTALLED_APPS 
```python
    INSTALLED_APPS = (
        ...
        'rest_framework_swagger',
    )
```
3、修改项目 urls.py，类似下面这样：
```python
from django.conf.urls import url
from rest_framework_swagger.views import get_swagger_view

schema_view = get_swagger_view(title='API 接口文档')

urlpatterns = [
    url(r'^docs$', schema_view)
]
```
本例中的效果如下所示：

![rest_framework_swagger](https://upload-images.jianshu.io/upload_images/12989993-ca869ecc9dd53b90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![rest_framework_swagger](https://upload-images.jianshu.io/upload_images/12989993-85e12d3f9e2eae48.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![交互](https://upload-images.jianshu.io/upload_images/12989993-ac1ec9aff9352063.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![交互](https://upload-images.jianshu.io/upload_images/12989993-e95ec77e1baaf1cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

功能和原生的大同小异，每个人喜欢的风格不一样，网上还有很多生成 api 文档的轮子，大家可以选一款自己喜欢的直接用就好。

本样例源完整代码已上传至百度云，微信公众号 somenzz 回复「api」获取下载链接，欢迎一起学习交流。

![个人微信公众号](https://upload-images.jianshu.io/upload_images/12989993-12f2c4e795cbeaf9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



（完）

加个人微信公众号 somenzz，及时获得最新原创技术干货，和你一起学习。

![个人公众号](/assets/img/wechat2.jpg)
