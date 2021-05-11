---
title: 给hexo搭建的github-page绑定域名
date: 2021-05-11 21:31:38
tags:
- hexo
- github
---

# github-page绑定域名

心血来潮，别人家的博客都有域名，我也要！

<img src="https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fdingyue.nosdn.127.net%2FzHz6HuOqltLyAdSJvjhSrQIcU5irErphu%3Dk3pPpXDOvK41516856240100compressflag.jpg&refer=http%3A%2F%2Fdingyue.nosdn.127.net&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1623332105&t=8140b449423b67261bfb0f4be4f7153f" alt="img" style="zoom:50%;" />

## 1. 注册域名

注册域名我选择的是阿里云的[万网](https://wanwang.aliyun.com/)，由于是新人，选择了一元购域名专区。

![image-20210511214823347](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210511214823347.png)

一元购，冲冲冲！！

![image-20210511215212606](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210511215212606.png)

支付时需要创建信息模板才可以支付，点击创建新的信息模板创建就OK啦。

创建信息模板后需要进行邮箱认证和实名认证。

域名注册完毕！

## 2. 给github-page添加CNAME解析。

我用的是hexo搭建的个人博客，所以以下针对hexo。

在博客根目录下的`source`目录下新建个文件CNAME，文件内容填写自己注册的域名。

**注意**：不写www

```
$ /blog/CNAME

ghj1998.top
```

将修改推送到github。

```bash
$ hexo clean
$ hexo d
```

## 3. 添加域名解析

![image-20210511220015320](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210511220015320.png)

在域名管理页面点击解析

![image-20210511220117898](https://raw.githubusercontent.com/ghj1998/image_repository/main/image-20210511220117898.png)

添加两条记录，主机记录一个填写@，一个填写www。

这样无论有没有www，都可以访问到你的域名。

记录类型选择CNAME。

记录值输入github-page的地址，即 `用户名.github.io`

OK，域名解析完毕。



**等待几分钟，就可以用域名访问你的github-page了。**

欢迎访问我的域名 ：www.ghj1998.top

---

**自己遇到的问题**

解析完域名，发现DNS仍然有问题，然后发现是自己没有实名认证🤣, 阿里云不给我解析DNS。

