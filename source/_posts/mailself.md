---
title: How to mail myself on Linux or Mac when a Job is Done
date: 2022-01-11 12:36:33
tags:
- Linux 
- Bash
categories:
- Tech
---

近期一直在星巴克坐着，远程登陆着家里的Linux做一些计算，由于时常需要远程登录
做些计算，更是时不时需要自己登陆上去查看计算结果，突然想到，如果可以在家里的
计算完成后，脚本自动给我推送一个通知，或是给我发个邮件提醒我已经计算完成了，
好像听上去很美好的样子。本着这个想法，我搜索了一下相关的信息，果然有这方面的
内容，自己觉得很有意思，就拿着一块记录一下具体的实现方法好了。但自己也不太常
用windows系统了，因此主要能在linux和mac下能跑通就可以啦，更何况如果可以发邮件 
的话，系统倒是无所谓了，当然如果mac能有接口直接进行通知，或者推送到我手机上，
就更好不过了。

<!-- more -->

## 发送邮件

### Ubuntu

#### 1. 依赖

1. 需要安装`postfix`和`mailutils`， 不过用`apt`安装的`mailutils`的时候会自动安装`postfix`，所以只安装`mailutils`也行。

```shell
#不放心的话可以先安装这个
#sudo apt install postfix
sudo apt install mailutils
```

2. 安装`postfix`的时候会出现一个对话框，`tab`才能移动到`ok`。直接选择`ok`和`internet site`去使用网络服务[SMTP: Simple Mail Transportation Protocol]就可以了

3. 名称随便设置就可以了

4. 这个设置对话框可以通过
   
   ```shell
   dpkg-reconfigure postfix
   ```
   
   重新进行配置

5. 此时已经可以通过`mail`命令发送邮件了，发出方会是`$USER $MACHINE` 这样，但是这样的话会被邮件系统认为是垃圾邮件，不方便进行查看，所以需要一个网络服务商进行邮件中转
   
   ```shell
   mail -s "topic" destination@xxx.com <<< str
   echo "string" | mail -s "topic" destination@xxx.com
   ```

#### 2. 配置postfix以使用gmail作为中转(Relay)【SMTP】

1. 可以通过`postconf`查看当前`postfix`的配置值

2. 首先修改中转服务器地址：

    ```
    sudo vim /etc/postfix/main.cf

    ```
    搜索到`relayhost =`，这个就是用来设置Gmail STMP域名
    ```
    ...
    relayhost = [smtp.gmail.com]:587
    mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
    ...
    ```

3. 配置sasl身份验证

    ```
    ...
    relayhost = [smtp.gmail.com]:587
    ...
    smtp_tls_CApath=/etc/ssl/certs
    #smtp_tls_security_level=may
    smtp_tls_security_level=encrypt
    smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache
    ...
    recipient_delimiter = +
    inet_interfaces = all
    inet_protocols = all
    smtp_sasl_auth_enable = yes
    smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
    smtp_sasl_security_options = noanonymous
    ```
    保存退出即可。这里进行的设置包括：
    `smtp_sasl_auth_enable`: 启用sasl身份验证
    `smtp_sasl_passwd_maps`: 设置用户名密码发送要邮件中转服务器
    `smtp_tls_security_level`: 加密传输过程
    `smtp_sasl_security_options`: SMTP安全选项 
        `noanonymous`:仅允许相互认证的方法

4. 设置sasl凭据，即用户名密码

    ```
    sudo vim /etc/postfix/sasl_passwd
    ```
    这部分设置自己的gmail账号密码用来验证，格式大概是这样的
    ```
    # destination credentials
    #[smtp.domain.name] username:password
    [smtp.gmail.com]:587 [email account]:password
    ```
    由于这部分内容比较重要，保护一下，所以限制一下文件的读取权限：
    ```
    sudo chown root:root /etc/postfix/sasl_passwd
    sudo chmod 600 /etc/postfix/sasl_passwd
    ```

5. 转换密码文件为数据库
```
postmap /etc/postfix/sasl_passwd
```

6. 检查设置
    
差不多已经完成了，最后检查一下配置是否有错误
```
postfix check
```
这里可以忽略这么一条警告：
```
postfix/postfix-script: warning: symlink leaves directory:
/etc/postfix/./makedefs.out
```

7. 重启`postfix`
最后重启`postfix`即可，差不多就完成了。

```
sudo systemctl restart postfix 
sudo systemctl status postfix
```

8. 测试邮件

```
echo "Test Postfix Gmail SMTP Relay" | mail -s "Postfix Gmail SMTP Relay" [email
protected]
```
以上是一个测试邮件，应该可以绕过邮件系统的垃圾邮件检测了！增加一个日志系统来检查
邮件的发送情况：

```
tail /var/log/mail.log
```

9. 错误：
如果存在错误：
```
...status=deferred (SASL authentication failed; server smtp.gmail.com[74.125.133.108] said: 535-5.7.8 Username and Password not accepted.
```
这个是gmail的身份验证，gmail默认开启的安全性较高的应用程序访问，这里可以修改。

[安全性较低的应用程序访问](https://myaccount.google.com/lesssecureapps)

10. 完成！
