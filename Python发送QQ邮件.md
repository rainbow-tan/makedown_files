Python发送QQ邮件

## 1、登陆QQ邮箱，获取授权码

可以参考 [官网说明](https://service.mail.qq.com/cgi-bin/help?subtype=1&&id=28&&no=1001256)

- 登录[QQ邮箱](https://mail.qq.com/)

![image-20220516130713724](https://img2022.cnblogs.com/blog/1768648/202205/1768648-20220516203918170-791747279.png)

- 点击设置

![image-20220516130856063](https://img2022.cnblogs.com/blog/1768648/202205/1768648-20220516203918478-1902101668.png)

点击账户、点击开启POP3/SMEP服务

![image-20220516130944697](https://img2022.cnblogs.com/blog/1768648/202205/1768648-20220516203918782-1514803690.png)

点击开启后验证密保，然后根据操作发送短信

![image-20220516131044710](https://img2022.cnblogs.com/blog/1768648/202205/1768648-20220516203919021-420958874.png)

![image-20220516131152989](https://img2022.cnblogs.com/blog/1768648/202205/1768648-20220516203919234-1203528879.png)

然后你就得到了你的授权码

## 2、发送文本邮件

发送一个只需要显示普通文字的邮件

```python
import datetime
import smtplib
import time
from email.mime.text import MIMEText


def main():
    email_host = "smtp.qq.com"
    email_port = 465
    email_sender = "1310693853@qq.com"
    password = 'xxxxxxxxxxxxxxxx'
    email_receiver = ["1150646501@qq.com"]
    email_cc = ["1150646502@qq.com"]

    body = """测试Python发送邮件【正文】
今天是 {}
现在的时间是 {}
测试正文完毕~~~~
""".format(datetime.datetime.today().strftime("%Y年%m月%d日"),
           time.strftime('%H:%M:%S', time.localtime()))

    msg = MIMEText(body, 'plain')
    msg["Subject"] = "邮件的主题"  # 邮件主题描述
    msg["From"] = email_sender  # 发件人显示,不起实际作用,只是显示一下
    msg["To"] = ",".join(email_receiver)  # 收件人显示,不起实际作用,只是显示一下
    msg["Cc"] = ",".join(email_cc)  # 抄送人显示,不起实际作用,只是显示一下

    with smtplib.SMTP_SSL(email_host, email_port) as smtp:  # 指定邮箱服务器
        smtp.login(email_sender, password)  # 登录邮箱
        smtp.sendmail(email_sender, email_receiver, msg.as_string())  # 分别是发件人、收件人、格式
        smtp.quit()
    print("发送邮件成功!")


if __name__ == '__main__':
    main()
```

邮件显示

![image-20221101154119559](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221101154119559.png)

## 3、发送HTML格式的邮件

```python
import datetime
import smtplib
import time
from email.mime.text import MIMEText


def main():
    email_host = "smtp.qq.com"
    email_port = 465
    email_sender = "1310693853@qq.com"
    password = 'xxxxxxxxxxxxxxxx'
    email_receiver = ["1150646501@qq.com"]
    email_cc = ["1150646502@qq.com"]

    body = """测试Python发送邮件【正文】
<H1>今天是 {}</H1>
<H2>现在的时间是 {}</H2>
<p style="
    color: tomato;
">测试正文完毕~~~~</p>
""".format(datetime.datetime.today().strftime("%Y年%m月%d日"),
           time.strftime('%H:%M:%S', time.localtime()))

    msg = MIMEText(body, 'html')
    msg["Subject"] = "邮件的主题-发送HTML格式的邮件"  # 邮件主题描述
    msg["From"] = email_sender  # 发件人显示,不起实际作用,只是显示一下
    msg["To"] = ",".join(email_receiver)  # 收件人显示,不起实际作用,只是显示一下
    msg["Cc"] = ",".join(email_cc)  # 抄送人显示,不起实际作用,只是显示一下

    with smtplib.SMTP_SSL(email_host, email_port) as smtp:  # 指定邮箱服务器
        smtp.login(email_sender, password)  # 登录邮箱
        smtp.sendmail(email_sender, email_receiver, msg.as_string())  # 分别是发件人、收件人、格式
        smtp.quit()
    print("发送邮件成功!")


if __name__ == '__main__':
    main()
```

邮件显示

![image-20221101154809161](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221101154809161.png)

## 4、发送带附件的邮件

```python
import datetime
import smtplib
import time
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText


def main():
    email_host = "smtp.qq.com"
    email_port = 465
    email_sender = "1310693853@qq.com"
    password = 'xxxxxxxxxxxxxxxx'
    email_receiver = ["1150646501@qq.com"]
    email_cc = ["1150646502@qq.com"]

    body_html = """测试Python发送邮件【正文】
    <H1>今天是 {}</H1>
    <H2>现在的时间是 {}</H2>
    <p style="
        color: tomato;
    ">测试正文完毕~~~~</p>
    """.format(datetime.datetime.today().strftime("%Y年%m月%d日"),
               time.strftime('%H:%M:%S', time.localtime()))

    msg = MIMEMultipart()
    msg.attach(MIMEText(body_html, 'html'))  # 添加HTML格式内容

    att = MIMEText(open('files/172.17.140.80-CPU使用率.png', 'rb').read(), 'base64', 'utf-8')
    att["Content-Type"] = 'application/octet-stream'
    att.add_header("Content-Disposition", "attachment", filename=("utf-8", "", "172.17.140.80-CPU使用率.png"))
    msg.attach(att)  # 添加附件

    with open('files/audit_auditrecord.json', 'rb') as f:
        json_content = f.read()
    att2 = MIMEText(json_content, 'base64', 'utf-8')
    att2["Content-Type"] = 'application/octet-stream'
    att2.add_header("Content-Disposition", "attachment", filename=("utf-8", "", "audit_auditrecord.json"))
    msg.attach(att2)  # 添加附件

    msg["Subject"] = "邮件的主题-带附件的邮件"  # 邮件主题描述
    msg["From"] = email_sender  # 发件人显示,不起实际作用,只是显示一下
    msg["To"] = ",".join(email_receiver)  # 收件人显示,不起实际作用,只是显示一下
    msg["Cc"] = ",".join(email_cc)  # 抄送人显示,不起实际作用,只是显示一下

    with smtplib.SMTP_SSL(email_host, email_port) as smtp:  # 指定邮箱服务器
        smtp.login(email_sender, password)  # 登录邮箱
        smtp.sendmail(email_sender, email_receiver, msg.as_string())  # 分别是发件人、收件人、格式
        smtp.quit()
    print("发送邮件成功!")


if __name__ == '__main__':
    main()
```

邮件显示

![image-20221101160725170](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221101160725170.png)



## 5、发送正文直接显示图片的邮件

```python
import datetime
import smtplib
import time
import uuid
from email.mime.image import MIMEImage
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText


def main():
    email_host = "smtp.qq.com"
    email_port = 465
    email_sender = "1310693853@qq.com"
    password = 'gquzhzcutxnnhjfd'
    email_receiver = ["1150646501@qq.com"]
    email_cc = ["1150646502@qq.com"]

    u = uuid.uuid4().hex  # 作为图片的唯一ID
    u2 = uuid.uuid4().hex  # 作为图片的唯一ID
    body_html = """测试Python发送邮件【正文】
    <H1>今天是 {}</H1>
    <H2>现在的时间是 {}</H2>
    <p style="
        color: tomato;
    ">测试正文完毕~~~~</p>
    <p><img src="cid:{}"></p>
    <p><img src="cid:{}" width="600px" height="600px"></p>
    """.format(datetime.datetime.today().strftime("%Y年%m月%d日"),
               time.strftime('%H:%M:%S', time.localtime()), u, u2)

    msg = MIMEMultipart()
    msg.attach(MIMEText(body_html, 'html'))  # 添加HTML格式内容

    with open('files/172.17.140.80-CPU使用率.png', 'rb') as f:
        content = f.read()
    img = MIMEImage(content)
    img.add_header('Content-ID', f'{u}')  # 定义图片 ID,在 HTML 文本中引用
    msg.attach(img)

    with open('files/172.17.140.120-CPU使用率.png', 'rb') as f:
        content = f.read()
    img = MIMEImage(content)
    img.add_header('Content-ID', f'{u2}')  # 定义图片 ID,在 HTML 文本中引用
    msg.attach(img)

    msg["Subject"] = "邮件的主题-正文直接显示图片的邮件"  # 邮件主题描述
    msg["From"] = email_sender  # 发件人显示,不起实际作用,只是显示一下
    msg["To"] = ",".join(email_receiver)  # 收件人显示,不起实际作用,只是显示一下
    msg["Cc"] = ",".join(email_cc)  # 抄送人显示,不起实际作用,只是显示一下

    with smtplib.SMTP_SSL(email_host, email_port) as smtp:  # 指定邮箱服务器
        smtp.login(email_sender, password)  # 登录邮箱
        smtp.sendmail(email_sender, email_receiver, msg.as_string())  # 分别是发件人、收件人、格式
        smtp.quit()
    print("发送邮件成功!")


if __name__ == '__main__':
    main()

```

邮件显示

![image-20221101163143399](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221101163143399.png)















































[参考链接](https://www.cnblogs.com/shenh/p/14267345.html)