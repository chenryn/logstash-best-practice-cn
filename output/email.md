# 发送邮件(Email)

## 配置示例

```
output {
    email {
        to => "admin@website.com,root@website.com"
        cc => "other@website.com"
        via => "smtp"
        subject => "Warning: %{title}"
        options => {
            smtpIporHost       => "localhost",
            port               => 25,
            domain             => 'localhost.localdomain',
            userName           => nil,
            password           => nil,
            authenticationType => nil, # (plain, login and cram_md5)
            starttls           => true
        }
        htmlbody => ""
        body => ""
        attachments => ["/path/to/filename"]
    }
}
```

## 解释

*outputs/email* 插件支持 SMTP 协议和 sendmail 两种方式，通过 `via` 参数设置。SMTP 方式有较多的 options 参数可配置。sendmail 只能利用本机上的 sendmail 服务来完成 —— 文档上描述了 Mail 库支持的 sendmail 配置参数，但实际代码中没有相关处理，不要被迷惑了。。。

