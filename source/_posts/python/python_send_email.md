---
title: 使用Python实现发送带附件的邮件
tags: [python,email]
date: 2017-12-16
categories: Python
---

#### 使用smtplib和email库发送邮件

* 使用`smtplib`库来发送邮件
* 使用`Email`库来处理邮件消息


#### Demo代码
``` python
#!/usr/bin/python
# -*- coding: UTF-8 -*-

import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from email.header import Header

import logging


class EMailOpr:
    def __init__(self):
        self.sender = '发送者邮箱'
        self.receivers = ['接受者邮箱1', "接受者邮箱2"]
        self.cc = ['抄送1', '抄送2']
        self.hide = ["暗抄1", "暗抄2"]
        self.smtp = 'smtp.****.com'
        self.smtp_port = 25
        self.user = "发送者账号"
        self.passwd = "发送者密码"

    # 发送邮件
    def send(self, subject, content="", att_path=""):
        """
        发送邮件
        :param subject: 邮件主题
        :param content: 邮件内容
        :param att_path: 邮件附件地址
        :return:
        """
        # 创建一个带附件的实例
        message = MIMEMultipart()
        message['From'] = self.sender
        message['To'] = ",".join(self.receivers)
        message['Cc'] = ",".join(self.cc)
        message['Subject'] = Header(subject, 'utf-8')

        # 邮件正文内容
        message.attach(MIMEText(content, 'plain', 'utf-8'))

        # 添加附件
        if att_path != "":
            # 构造附件1
            att1 = MIMEText(open(att_path, 'rb').read(), 'base64', 'utf-8')
            att1["Content-Type"] = 'application/octet-stream'
            # 这里的filename可以任意写，写什么名字，邮件中显示什么名字
            att1["Content-Disposition"] = 'attachment; filename=%s' % att_path[att_path.rfind("/")+1:]
            message.attach(att1)

        try:
            smtpObj = smtplib.SMTP()
            smtpObj.connect(self.smtp, self.smtp_port)
            smtpObj.login(self.user, self.passwd)
            smtpObj.sendmail(self.sender, self.receivers+self.cc+self.hide, message.as_string())
            print u"邮件发送成功"
        except smtplib.SMTPException:
            print u"Error: 无法发送邮件"

if __name__ == '__main__':
    file_name = "附件文件路径"
    email = EMailOpr()
    email.send("主题", "内容", file_name)

```
