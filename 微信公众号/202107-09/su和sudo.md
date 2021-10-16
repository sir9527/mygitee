

* Linux的su和sudo
    * 1.mac终端登陆远程服务器
    * 2.su命令
    * 3.sudo命令

# Linux的su和sudo

## 1.mac终端登陆远程服务器

ssh -p 22 root@82.157.112.153                         

ssh -p 22 test_xjq@82.157.112.153


## 2.su命令

 `switch user`：切换用户

- 语法：su - <user_name>
- 常用：带参数"-"不仅切换到用户，还会切换到用户的环境变量以及各种设置
- 常用命令：
  - 切换到root用户 "su - "  需要输入root用户密码;  
  - 切换到root用户 "su"     需要输入root用户密码 ;
  -  切换到普通用户 "su test_xjq"



## 3.sudo命令

`super user do`：以超级用户（root 用户）的方式执行命令

- sudo su -  ：切换到root用户。需要输入当前用户密码
- sudo 命令 ：使用超级管理员权限进行执行命令
- sudo命令相比su命令的优点：可以避免暴露root用户的密码，同时给用户提供root权限
- 注：sudo命令需要root用户在"/etc/sudoers"中添加权限























