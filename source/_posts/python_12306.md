---
layout: post
title: "Python-骚操作-抢火车票"
date: 2019-01-06 01:38:57
comments: true
reward: true
tags: 
	- Python
	- 爬虫
---


还有不到一个月就过春节了，你回家的火车票都买了吗？如果没有买到的话，不妨试用下本文的 Python 程序来使用 Python 帮你抢火车票，也可以帮你的家人和朋友来抢票，顺带学习一下 Python 爬虫技术，可谓一举两得，何乐而不为？

<!-- more -->

我本来想自己写一个练练手的，但是转眼一想，Python 本身最大的优势是什么，不就是有很多牛逼的人已经造好轮子了吗？你只需要知道这些轮子并会使用就行了，这样会节省你大量的精力和时间，而且站在巨人的肩膀上，会看得更远。于是我在 github 上一搜索，果然有不少抢票程序，有的是 Python2，有的是 Python3，按 start 数据排序，经过亲自使用和对比，我选择了一个相对较好用的程序，并稍加以改进和完善。

项目 github 地址：https://github.com/xiaoshun007/12306Python。

在此感谢作者 xiaoshun007 的分享。

项目简介：hack12306.py 是一个 Python 3.x 版的[12306.cn](http://www.12306.cn/mormhweb/)自动订票程序。利用[splinter](https://splinter.readthedocs.io/en/latest/index.html)（一个开源的用来通过python自动化测试web的工具），让电脑自动操作网页。支持的功能：
    1、支持配置出发地、目的地、乘车日
    2、支持配置车次类型（动车、高铁等）
    3、支持配置出发时间
    4、需要手动输入登录验证码
    5、支持配置预定车次的选择顺序（使用 order 字段配置，数字0：从上至下选择；数字x（1、2、3、4...）：车次从上到下的序号，配置2表示列表中的第二个车次）
    6、支持预定、购票自动完成	
    7、支持配置文件路径指定
    8、支持席别指定
    9、支持是否允许分配无座   
还不支持的功能：
    1、邮件提醒

于是，我在此基础上，**加入邮件提醒的功能，并修复一些小 bug，公众号后台回复关键字【12306】获取我完善后的抢票程序源码，再按下方的步骤来操作即可**。程序的流程图如下：
![流程图.jpg](https://upload-images.jianshu.io/upload_images/12989993-5e757817bfe6e197.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 步骤一、环境准备
1、安装 chromedriver
由于程序使用 chrome 浏览器，因此需要安装 chromedriver，其实很简单，就是下载自己电脑上 chrome 浏览器对应的 chromedriver 即可，网上bing 一下就找到了，也可参考之前的文章[Python 云端学习](https://www.jianshu.com/p/385fe9c904be) 中 chromedriver 的安装方法。我分享在网盘里 chromedriver 对应的 chrome 浏览器版本为 71.0.3578.98，不过也没有那么严格，只要是较新的  chrome 浏览器都可以使用我提供的这个 chromedriver 。将 chromedriver  放在一个你想放置的目录下，这个路径需要配置在配置文件中。
![](https://upload-images.jianshu.io/upload_images/12989993-81a3ecc3e9f9e9e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



2、安装依赖的 Python 三方库
安装自动化工具库 splinter 和 邮件发送模块 zmail 。直接命令行执行
```python
pip install splinter
pip install zmail
```

## 步骤二、修改配置文件
配置文件 config.ini 需要修改以下几个地方：
1、你的12306账号、密码
```python
## 登陆账号和密码
[login]
### username：12306登录用户名，必选参数
username= 填写你的12306用户名
### password：12306登录密码，必选参数
password= 填写你的12306密码
```
2、你要买票的始发站，终点站，日期

```python
## cookie信息，出发站，目的站
[cookieInfo]
### starts：对应搜索框出发地，必选参数，请输入中文名称，例如：武汉
starts=苏州
### ends：对应搜索框目的地，必选参数，请输入中文名称，例如：南京
ends=信阳
### dtime：对应搜索框出发日，必选参数，时间格式：年-月-日，例如 2018-01-19
## 时间格式2018-01-19
dtime=2019-02-02
```
3、你要为其买票的人姓名
```python
## users：乘客姓名，必选参数，中文姓名，支持多个乘客，用英文逗号隔开，例如：张三,李四
### 乘客姓名需要提前加入到登录的12306账号的联系人中，为了程序自动选择乘客姓名
[userInfo]
users = 郑征 
```

4、chromedirver 的路径
```python
## 路径信息
[pathInfo]
### driver_name: 浏览器名称，必选参数
driver_name = chrome
### executable_path: 浏览器驱动路径，必选参数
### windows路径例如：C:\Users\sanshunfeng\Downloads\chromedriver.exe
executable_path = E:\GitHub\python\pachong\tools\chromedriver.exe
```
5、发送邮件的配置信息
```python
[mail]
mail_user = 你的邮箱如 ：somenzz@163.com
mail_pwd = 你的密码
receiver = 你的收件地址：如 somenzz@163.com
```

其他如要买车次类型，几等座，顺序号等参考配置文件的注释进行修改即可 ，大多数人使用默认的配置就够了。

## 步骤三、运行源代码
直接在命令行执行

```python
python hack12306.py
```
即可自动读取配置文件并运行自动抢票程序。

**代码修改说明：**

1、手工确认登陆成功。程序在登陆12306网站后，12306可能会跳转到出现问题的报错页面，提示“网络可能出现问题的页面”（可能是一种反爬虫措施），此时程序将陷入无限等待。为防止此种情况发生，我这边将将自动检查登陆结果的程序替换为手工检查，点击验证码登陆后，请在命令行界面输入 “Y”，即可使程序继续运行，这个修改是通用的，不论是否跳出网络错误页面均可运行。

```python
# 验证码需要自行输入，程序自旋等待，直到验证码通过，点击登录
# 为防止跳转错误页面陷入死待，此处改为手工确认。
confirm = input("完成验证：Y/N: ")
if confirm == 'Y' or confirm == 'y':
    return
else:
    #输入其他值，程序退出
    exit(0)
# while True:
#     if self.driver.url != self.initmy_url:
#         sleep(1)
#     else:
#         break
```
上述注释掉的代码为修改前的代码。

2、邮件发送功能。
增加以下函数发送提醒邮件。
```python
def sendmail(self,subject,message):
    # 你的邮件内容
    mail_content ={
        'subject':subject,  # 邮件标题写在这
        'content_text':message,  # 邮件正文写在这
    }
    # 使用你的邮件账户名和密码登录服务器
    server = zmail.server(self.mail_user, self.mail_pwd)
    # 发送邮件指令
    server.send_mail([self.receiver], mail_content)
```
3、调整等待时间。有些时候由于网络延迟某些按钮等元素还未加载出来就被程序发送了点击命令，此时会报错，通过适当延长等待时间可以解决这个问题，就是调节代码中的 time.sleep(n) 中的 n 的值，如下图所示：
![调整等待时间](https://upload-images.jianshu.io/upload_images/12989993-665e9dee9eec3363.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4、运行结果。
程序运行后会自动打开12306的页面登陆，并自动输入配置文件中的用户名和密码，点击验证码登陆后，在后台命令窗口输入Y，然后就可以看到浏览器在不停止的查询余票信息，当有符合条件的车票时将自动下单，并邮件通知。**如果第一次运行后报错了，那么请重试一次，一般第二次就不报错了。**
![Snipaste_2019-01-05_17-45-37.png](https://upload-images.jianshu.io/upload_images/12989993-9a5e9c0a8f47f246.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

后台打印的信息如下所示：
```python
===========hack12306 begin===========
映射出发地、目的地...
加载配置文件...

DevTools listening on ws://127.0.0.1:58067/devtools/browser/4426bbf5-49ca-439a-b73e-9217ececf3ea
开始登录...
等待验证码，自行输入...
完成验证：Y/N: Y
购票页面开始...
--------->选择的车次类型 D-动车
--------->选择的车次类型 GC-高铁/城际
--------->选择的发车时间 00:00--24:00
循环点击查询... 第 1 次
--------->选择的车次类型 D-动车
--------->选择的车次类型 GC-高铁/城际
--------->选择的发车时间 00:00--24:00
循环点击查询... 第 2 次
--------->选择的车次类型 D-动车
--------->选择的车次类型 GC-高铁/城际
--------->选择的发车时间 00:00--24:00
循环点击查询... 第 3 次
--------->选择的车次类型 D-动车
--------->选择的车次类型 GC-高铁/城际
--------->选择的发车时间 00:00--24:00
循环点击查询... 第 4 次
--------->选择的车次类型 D-动车
--------->选择的车次类型 GC-高铁/城际
--------->选择的发车时间 00:00--24:00
循环点击查询... 第 5 次
--------->选择的车次类型 D-动车
--------->选择的车次类型 GC-高铁/城际
--------->选择的发车时间 00:00--24:00
循环点击查询... 第 6 次
......
```
5、关于12306的验证码。
这验证码可以说是无敌了，连人有时侯都难以分辨。不过仍有人破解这个验证码，准确率可以说是相当高了，可以点击下面的链接了解详情。
https://github.com/andelf/fuck12306


6、生成windows可执行程序。
如果你想让自己的Python程序发给不懂Python的人使用，还是编译成 exe 发给他们好用，省得安装各种依赖包。这里说下如何将 python 源文件编译为 exe 文件。工具有很多，坑也很多，不建议过多研究，作为学习者直接运行源代码妥妥的。这里使用 pyinstaller。先安装打包工具：

```python
pip install pywin32
pip install PyInstaller
```
在源代码所在的目录下执行命令：

```python
pyinstaller -F hack12306.py
```
等待完成即可在 dist 目录找到可执行的 exe 文件。

如果代码使用了第三方库，则需要将第三方库包也放在源代码所在的目录，如本例中的：
![](https://upload-images.jianshu.io/upload_images/12989993-7a5d4d078d553548.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


7、附部分源码：

```python
    def selUser(self):
        print(u'开始选择用户...')
        for user in self.users:
            self.driver.find_by_text(user).last.click()

    def confirmOrder(self):
        print(u"选择席别...")
        if self.seatType:
            self.driver.find_by_value(self.seatType).click()
        else:
            print(u"未指定席别，按照12306默认席别")

    def submitOrder(self):
        print(u"提交订单...")
        sleep(1)
        self.driver.find_by_id('submitOrder_id').click()

    def confirmSeat(self):
        # 若提交订单异常，请适当加大sleep的时间
        sleep(2)
        print(u"确认选座...")
        if self.driver.find_by_text(u"硬座余票<strong>0</strong>张") == None:
            self.driver.find_by_id('qr_submit_id').click()
        else:
            if self.noseat_allow == 0:
                self.driver.find_by_id('back_edit_id').click()
            elif self.noseat_allow == 1:
                self.driver.find_by_id('qr_submit_id').click()

    def buyTickets(self):
        t = time.clock()
        try:
            print(u"购票页面开始...")

            # 填充查询条件
            self.preStart()

            # 带着查询条件，重新加载页面
            self.driver.reload()

            # 预定车次算法：根据order的配置确定开始点击预订的车次，0-从上至下点击，1-第一个车次，2-第二个车次，类推
            if self.order != 0:
                # 指定车次预订
                self.specifyTrainNo()
            else:
                # 默认选票
                self.buyOrderZero()
            print(u"开始预订...")

            sleep(1)
            # 选择用户
            self.selUser()
            # 确认订单
            self.confirmOrder()
            # 提交订单
            self.submitOrder()
            # 确认选座
            self.confirmSeat()
            # 发送邮件
            self.sendmail("抢到票了","请及时付款")

            print(time.clock() - t)

        except Exception as e:
            print(e)

```

（完）

加个人微信公众号 somenzz，及时获得最新原创技术干货，和你一起学习。

![个人公众号](/assets/img/wechat2.jpg)
