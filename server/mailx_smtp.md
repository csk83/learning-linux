# 云服务器用mailx smtp ssl 发送邮件

## 1. 安装 mailx
`yum  install  mailx -y`

## 2. 修改配置
mailx 的配置文件为 /etc/mail.rc 修改让其通过 SMTP 发送邮件
```
set from=xxxx@qq.com
set smtp=smtp.qq.com             // smtp 服务器地址 企业邮箱是 smtp.exmail.qq.com
set smtp-auth-user=xxx@qq.com    // 用户名 一般为邮箱全地址
set smtp-auth-password=password  // 邮箱的密码，部分邮箱为授权码
```
正常情况自己的服务器一般就可以发送邮件了（云服务器一般不行，防火墙端口限制）。
## 3. 起开 SSL
### 生成证书
```
mkdir -p /root/.certs                                        ###    证书存放位置
cd  /root/.certs
###下面这几条命令每一条不是太明白，只知道生成了证书和证书文件，和启用的证书！
echo -n |openssl s_client -connect smtp.exmail.qq.com:465 |sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > qqexmail.crt
certutil -A -n "GeoTrust SSL CA" -t "C,," -d ~/.certs -i ~/.certs/qqexmail.crt  ### 这条可以不要
certutil -A -n "GeoTrust Global CA" -t "C,," -d ~/.certs -i ~/.certs/qqexmail.crt
certutil -A -n "GeoTrust SSL CA - G3" -t "Pu,Pu,Pu" -d ~/.certs/./ -i qqexmail.crt
certutil -L -d /root/.certs
```
### 修改配置

```
set from=xxxooo@xxx.com
set smtp=smtps://smtp.exmail.qq.com:465
set smtp-auth-user=xxxooo@xxx.com
set smtp-auth-password=你的授权码
set smtp-auth=login
set ssl-verify=ignore
set nss-config-dir=/root/.certs
```
## 4. 测试发送邮件的三种方法
- 命令行: mail -s "邮件标题" xxxooo@xxx.com,回车后输入内容按Ctrl+D发送邮件.
- 管道符: echo "邮件内容" | mail -s "邮件标题" xxxooo@xxx.com
- 文件内容作为邮件内容: mail -s "邮件标题" xxxooo@xxx.com < /tmp/t.txt

## 5. 参考资料
- [centos7 利用mailx发送邮件](https://www.cnblogs.com/liutao97/p/8387244.html "centos7 利用mailx发送邮件")
- [mailx及sendEmail的基本用法比较](https://blog.51cto.com/irow10/1812638 "mailx及sendEmail的基本用法比较")
- [在阿里云上无法使用mailx发送邮件的解决办法，验证可用](https://www.cnblogs.com/yeyu1314/p/10167944.html "在阿里云上无法使用mailx发送邮件的解决办法，验证可用")
