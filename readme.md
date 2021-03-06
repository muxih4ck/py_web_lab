<p align="center">
  <img src="https://seriousseverity.files.wordpress.com/2015/01/topguntocat.png?w=880&h=400&crop=1" width="auto" height="auto" /><br><h1>py_web_lab</h1><br><神奇的Python Web实验室></br>
</p>

## 关于 py_web_lab
**py_web_lab**是为了让木犀新人更好的学习Python web开发而打造的一系列实验, 分为**底层篇**和**应用篇**2个大实验, 每个大实验由若干个目标明确、给出资源和代码的小实验组成. <br/>

## 为什么要设计实验?
木犀一直崇尚**项目驱动的学习方式**, 但是从这2年的实际来看, 新人在做第一个项目的时候, 往往并不清楚如何规划项目, 如何一步步实现项目. 而通过实验的形式, 将一个项目划分为特定的目标明确的小步骤, 既让新人知道要如何一步步走, 同时每完成一个实验让新人感到有成就感而不至于在项目中迷失自己, 丧失兴趣(说的有点严重..嘎哈哈). <br/>
当然作为实验的设计者, 从宏观上规划和划分一个项目, 既考虑整体又不失细节, 是对能力提高, 对于木犀的老人是极为收益的😎 . <br/>

## 实验结构
### 实验零: 准备篇

> 实验目标: 阅读Flask源码, 弄清Flask的工作原理

+ [Flask源码分析](https://github.com/muxih4ck/py_web_lab/blob/master/src/flask/readme.md)
    + [Werkzeug与WSGI](https://github.com/muxih4ck/py_web_lab/blob/master/src/flask/wsgi.md)

### 实验一: 底层篇

> 实验目标: 从WSGI开始, 实现一个自带前端模版, 可以处理HTTP请求和简单sql查询的Web框架muxi

+ **Lab0: 神奇Web在哪里**
    - 内容: Web, http, 浏览器, 服务器, Web框架
    - 目标: 熟知[浏览器向服务器发请求建立http连接到返回响应的全过程], 了解Web框架的作用
    - 难度: lv0
+ **Lab1: "你的应用, 不!是你的应用" 之Python应用和Web服务器间的接口: WSGI**
    - 内容: WSGI, WSGI-Server, WSGI-APP, WSGI-Middleware
    - 目标: 理解WSGI协议, 知道Python Web框架处于WSGI协议的哪一部分
    - 难度: lv1
+ **Lab2: 我心中的框架, 你长什么样?**
    - 内容: 框架接口设计: 请求处理接口、前端模版接口、sql查询接口
    - 目标: 知道我们要做的框架是什么样子的, 思考如何让我们的框架变成这个样子
    - 难度: lv0
+ **Lab3: show time! 实现WSGI**
    - 内容: 实现WSGI APP核心类, 创建请求环境、获取请求、分发处理请求、处理响应
    - 目标: 浏览器中输入url、处理不同的http请求, 得到text/plain响应
    - 难度: lv4
+ **Lab4: 让响应更好看: 实现前端模版**
    - 内容: 实现一个类似Jinja的前端模版, 并集成到已实现的WSGI框架中
    - 目标: 浏览器中输入url、处理不同的http请求, 得到text/html响应
    - 难度: lv5
+ **Lab5: 毛主席说过:没有数据持久化的后端不是好后端, 向muxi添加sql支持!**
    - 内容: 实现sql operate cursor
    - 目标: 处理简单的sqlite数据库创建、查询、更新、删除sql语句
    - 难度: lv5
+ **Lab6: 是时候展现muxi的实力了: 用muxi框架开发一个简单博客应用**
    - 内容: 用先前lab实现的muxi框架写一个简单的博客
    - 目标: 😊 博客可以正常工作
    - 难度: lv5

### 实验二: 应用篇
还没想好构建什么应用orz.. 要包含爬虫、数据库(新人向最好是sql)、缓存、API、权限系统、登录验证、TokenAuth、 发送邮件、定时任务, 部署用docker和docker-compose. <br/>
感觉之前那个同学证件照爬虫就是一个不错的选择😂 , 有爬虫, 有API, 可以加上用户权限比如有些女孩子(好吧,其实是男孩子)的照片只有管理员(也就是我)可以看到, 然后可以给照片点赞, 定时任务可以做定时爬虫, 或者每晚定时发送好看的照片到邮箱(QQ)😄

## CopyRight

<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">Creative Commons Attribution-NonCommercial 4.0 International License</a>.
