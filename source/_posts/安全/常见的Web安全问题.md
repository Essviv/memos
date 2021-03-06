---
title: 常见的Web安全问题
author: essviv
date: 2016-10-14 10:20:54+0800
---

# 常见的WEB安全问题

## 1. XSS

跨站脚本攻击， 解决方式就是对所有的用户进行转义编码，永远不要相信用户的输入

## 2. CSRF

跨站请求伪造，原理图如下. 解决方式是给每个请求增加攻击者无法猜测的随机量，在接收到请求时，先校验这个随机量的有效性.

![csrf](https://github.com/Essviv/images/blob/master/csrf.jpg?raw=true)

## 3. 固定session攻击

会话固定攻击. 攻击者先登录欲攻击的网站，获取该会话的session, 并把获取到的session做为登录链接的一个参数，将构造的链接发给无辜的用户，用户点击链接登录成功后，在会话过期之前，攻击者就可以使用该session进行操作，获取登录用户的所有权限.解决方式是登录成功后，给用户重新发放一个session，废弃登录时的session.

## 4. 越权访问

* 横向越权：访问链接中带有一些ID标识，比如prdId = 5等等，攻击者就可以通过更改ID编码访问其它用户的资料，解决方式是尽量避免从前端获取相应的ID，如果无法避免，必须在后台增加相应的判断

* 纵向越权： 攻击者获取需要更高访问权限的链接，直接访问，解决方式是通过安全框架进行统一权限控制

## 5. 文件存储

文件存储最主要的问题有几个，

* 限制文件大小，避免允许用户上传不限大小的文件

* 用户上传文件的目录要关闭相应的执行权限，毕竟用户上传的文件内容是不可知的

## 6. 短信轰炸、邮件轰炸

对于短信验证码这类会给用户下发短信、邮件等操作，允许通过验证码、限制次数等方式进行人机实验，确定发起操作的不是攻击脚本，避免攻击者通过脚本短时间内发起大量操作，给用户造成困扰.

## 7. XFS

跨框架攻击，通过iFrame的隐藏特性，在正常操作的背后隐藏相应的iFrame，执行一些不可告人的操作, 防御的措施是在web服务器上配置相应的策略 