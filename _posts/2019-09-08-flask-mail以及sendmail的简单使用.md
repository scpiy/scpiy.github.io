---
layout:     post
title:      flask-mail以及sendmail的简单使用
subtitle:   linux, flask
date:       2019-09-08
author:     scpi
header-img: img/post-2019-09-08.jpg
catalog:	true
tags:
    - linux
    - flask
---



## flask-mail以及sendmail的简单使用

关于如何在ubuntu的终端中发送邮件以及如何在flask中使用flask-mail发送邮件。

这里是[**The-Flask-Mega-Tutorial-zh**](https://github.com/luhuisicnu/The-Flask-Mega-Tutorial-zh)的第七章，开始让web app使用邮件服务发送错误，之后也会用到邮件服务，至于是干什么，忘了。。。

**flask发送邮件是不用sendmail的，我刚开始不知道，折腾了好一会，虽然现在知道不用了，但我还是记下来吧，说不定恰好有人会有需要？~~总不能让我的时间白白浪费了吧。。。~~，以后说不定我也还会用到，姑且先记着。但是如果你对这个没兴趣，可以直接跳过。看旁边的小标题哦。**

首先让我发发牢骚，我跟着国人的教程走了好几个流程，改过来改过去，也发送不了，然后看英文结果第一个不到10分钟的时间就弄好了。不过也有可能是我太笨。

首先按照[Install and configure Sendmail on Ubuntu](https://gist.github.com/adamstac/7462202)这里的方法，安装并配置你的sendmail(想要访问前面那个网址似乎需要一些科学手段)。我自己觉得应该不会看不懂吧？就不翻译了。

1. If sendmail isn't installed, install it: `sudo apt-get install sendmail`
2. Configure `/etc/hosts` file: `nano /etc/hosts`
3. Make sure the line looks like this: `127.0.0.1 localhost yourhostname`
4. Run Sendmail's config and answer 'Y' to everything: `sudo sendmailconfig`
5. Restart apache `sudo service apache2 restart`

然后，如果一切顺利，在你的终端里：

```
sendmail youraddress@example.com               
Subject: Test Send Mail
Hello mail。
# ctrl + d结束正文部分，稍等不到一分钟，你就可以收到邮件了，不过应该是在垃圾邮件里。
```

`sendmail`是一个命令，之后跟的是你的邮箱地址，qq邮箱啊，163邮箱啊，gmail啊，应该都可以的(因为我只用网易邮箱做了测试，但至少网易邮箱是可行的。)有图有真相。

[![n1y1j1.md.png](https://s2.ax1x.com/2019/09/07/n1y1j1.md.png)](https://imgchr.com/i/n1y1j1)

虽然是垃圾邮件就是了，但至少发送成功了。



sendmail的比较常用的用法可以看这里[How do I use Sendmail?](https://help.dreamhost.com/hc/en-us/articles/216687518-How-do-I-use-Sendmail-)，上面那个例子也出于这里，我把文章大概看了看，它提供的功能我都还用不到诶，好像是密抄和什么其他的，中文我都不知道是什么的东西，暂时也没兴趣知道，还有在shell script中使用sendmail，很诱人，可是我还不会shell编程。。暂时跳过。

---

#### 在flask app里使用邮件服务。

比较尴尬的是，logging.handlers.SMTPHandler似乎只支持TSL。。

~~其实是因为我看不懂教程里面说的是什么东西什么的我才不会说出来呢！~~

总之，我选择使用flask-mail插件+163网易邮箱来发送错误信息。qq邮箱与国内其他邮箱应该类似，至少qq邮箱和163邮箱应该都是使用SSL连接的，至于其他的按实际情况为准，这里我以网易邮箱为例。

首先进行flask-mail的初始化，在`__init__.py`文件里

```python
from flask import Flask
from flask_mail import Mail

...
app = Flask(__name__)
mail = Mail(app)
```

然后是配置，我跟着教程走的，配置放在一个单独的文件`config.py`里。

```python
MAIL_SERVER = 'smtp.163.com'
MAIL_PORT = 465
# 划重点，选对，不知道自己是哪个的上网查查
MAIL_USE_TLS = False
MAIL_USE_SSL = True
# 划重点，这个密码。并不是登录密码，而是认证码，如果你不清楚，后面会说
MAIL_PASSWORD = '**********'
MAIL_USERNAME = '********@163.com'
```

[![n3MdFU.md.png](https://s2.ax1x.com/2019/09/08/n3MdFU.md.png)](https://imgchr.com/i/n3MdFU)

当你第一次来到这个页面，选择开启SMTP服务时，会提醒你需要设置"认证码"，自行设置好以后，上面的password就是这个认证码，而非你的登录密码，在这个页面你也能看到你使用的邮件的"server"，以及你是SSL连接。

具体的配置、初始化、或者如果你想更深入了解flask-mail，建议看看[flask-mail中文文档](http://www.pythondoc.com/flask-mail/)，虽说是中文，但其实翻译的并不全，但是对一般人应该够用。

然后就是在你的web app里实现发送邮件的服务(这个也可以参考上面的文档链接)。

我的错误处理函数集中在`app/errors.py`这个文件里。

```python
from flask import render_template
# 这个db不用管，这个源码也只是看个思路就好，有不明白的地方也没什么
from app import app, db, mail
from flask_mail import Message


def send_error_message(error_message):
    # send error message
    if not app.debug:
        # 因为我在配置里设置了MAIL_DEFAULT_SENDER，所以没有填写发件人。
        msg = Message(error_message, recipients=['h1273396866@163.com'])
        msg.body = error_message
        msg.html = f"<b>{error_message}</b>"
        mail.send(msg)



@app.errorhandler(404)
def not_found_error(error):
    send_error_message('404 error occur')
    return render_template('404.html'), 404


@app.errorhandler(500)
def internal_error(error):
    send_error_message('500 error occur')
    db.session.rollback()
    return render_template('500.html'), 500

```

上面就是我的源码，很~~简陋~~但很**实用**哦。

简单来说就是定义一个接受错误信息的函数`send_error_message`，当调用错误处理函数时获得错误信息并使用获得的错误信息调用`send_error_message`，先发送邮件，再进行其他的错误处理，当然也可以先进行其他的错误处理，在发送邮件，效果大概一样。

*如果你想要获取更多的错误信息，可以尝试使用`traceback`这个模块，其方法`traceback.format_exc()`会返回与终端相同的错误信息，将这个作为mail的正文就可以获得错误信息了，虽然不是很好看。*

flask-mail发送邮件真的很简单，[flask-mail中文文档](http://www.pythondoc.com/flask-mail/)这里面说的很清楚很明白也很简单，我就直接复制粘贴吧，免得有的人不想在几个页面之间跳来跳去。

> ## 配置 Flask-Mail
>
> **Flask-Mail** 使用标准的 Flask 配置 API 进行配置。下面这些是可用的配置型(每一个将会在文档中进行解释):
>
> - **MAIL_SERVER** : 默认为 **‘localhost’**
> - **MAIL_PORT** : 默认为 **25**
> - **MAIL_USE_TLS** : 默认为 **False**
> - **MAIL_USE_SSL** : 默认为 **False**
> - **MAIL_DEBUG** : 默认为 **app.debug**
> - **MAIL_USERNAME** : 默认为 **None**
> - **MAIL_PASSWORD** : 默认为 **None**
> - **MAIL_DEFAULT_SENDER** : 默认为 **None**
> - **MAIL_MAX_EMAILS** : 默认为 **None**
> - **MAIL_SUPPRESS_SEND** : 默认为 **app.testing**
> - **MAIL_ASCII_ATTACHMENTS** : 默认为 **False**
>
> 另外，**Flask-Mail** 使用标准的 Flask 的 `TESTING` 配置项用于单元测试(下面会具体介绍)。
>
> 邮件是通过一个 `Mail` 实例进行管理:
>
> ```
> from flask import Flask
> from flask_mail import Mail
> 
> app = Flask(__name__)
> mail = Mail(app)
> ```
>
> 在这个例子中所有的邮件将会使用传入到 `Mail` 实例中的应用程序的配置项进行发送。
>
> 或者你也可以在应用程序配置的时候设置你的 `Mail` 实例，通过使用 **init_app** 方法:
>
> ```
> mail = Mail()
> 
> app = Flask(__name__)
> mail.init_app(app)
> ```
>
> 在这个例子中邮件将会使用 Flask 的 `current_app` 中的配置项进行发送。如果你有多个具有不用配置项的多个应用运行在同一程序的时候，这种设置方式是十分有用的，
>
> ## 发送邮件
>
> 为了能够发送邮件，首先需要创建一个 `Message` 实例:
>
> ```
> from flask_mail import Message
> 
> @app.route("/")
> def index():
> 
>     msg = Message("Hello",
>                   sender="from@example.com",
>                   recipients=["to@example.com"])
> ```
>
> 你能够设置一个或者多个收件人:
>
> ```
> msg.recipients = ["you@example.com"]
> msg.add_recipient("somebodyelse@example.com")
> ```
>
> 如果你设置了 `MAIL_DEFAULT_SENDER`，就不必再次填写发件人，默认情况下将会使用配置项的发件人:
>
> ```
> msg = Message("Hello",
>               recipients=["to@example.com"])
> ```
>
> 如果 `sender` 是一个二元组，它将会被分成姓名和邮件地址:
>
> ```
> msg = Message("Hello",
>               sender=("Me", "me@example.com"))
> 
> assert msg.sender == "Me <me@example.com>"
> ```
>
> 邮件内容可以包含主体以及/或者 HTML:
>
> ```
> msg.body = "testing"
> msg.html = "<b>testing</b>"
> ```
>
> 最后，发送邮件的时候请使用 Flask 应用设置的 `Mail` 实例:
>
> ```python
> mail.send(msg)
> ```

如果顺利，就运行下自己的app，然后触发一个500或者404错误，去看看是否收到了邮件？反正我是收到了诶。

---

那么今天就到这里了，感谢你的观看。

如果有什么不明白的地方欢迎与我联系。