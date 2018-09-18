---
layout: post                  
title: "Ajax Session"             
date: 2018-09-18               
tag:  JavaWeb
---

## Session与Cookie

### session原理：

 http是无状态的协议，客户每次读取web页面时，服务器都打开新的会话，而且服务器也不会自动维护客户的上下文信息，session就是一种保存上下文信息的机制，针对每一个用户，session的内容在服务器端，通过sessionId来区分不同的用户，session是以cookie或url重写为基础的，默认用cookie实现，系统会创造一个JSESSIONID的输出cookie，我们称为session cookie，以区分persistent cookies，session cookie 是存储在浏览器内存中的，并不是写到硬盘上；我们通常是看不到JSESSIONID的，但当我们禁用浏览器的cookie后，web服务器就会采用url重写的方式传递sessionid，session cookie针对某一次会话而言，会话结束session cookie也就消失了。

### **session cookie的区别**

1. session保存在服务器，客户端不知道其中的信息，cookie保存在浏览器，服务器可以知道其中信息
2. session中保存的是对象，cookie保存的是字符串
3. session不区分路径，同一个用户在访问一个网站期间，所有的session在任何一个地方都能访问到；而cookie中如果设置了路径参数，那么同一个网站中不同路径下的cookie互相是访问不到的

### session与cookie的联系

session需要借助cookie才能正常工作，如果客户端完全禁用cookie，session将失效

### session与cookie的应用

实现自动登录：在用户访问某个网站并注册后，就会收到一个唯一用户ID的cookie，客户再次重新连接的时候，这个用户ID会自动返回，服务器对它进行检查，确定他是否为注册用户选择了自动登录，从而使用户无需明确给出用户名和密码，就可以访问服务器端资源

### 会话跟踪

通常session cookie是不能跨窗口使用的，当新打开一个浏览器窗口进入相同的页面的时候，系统会赋予你一个新的sessionid，这样我们信息共享的目的就达到了，此时我们可以先把sessionid保存在persistent cookie中，然后在新窗口读出来，就可以得到上一个窗口的sessionid，这样通过session cookie和persistent cookie的结合就实现了跨窗口的session tracking

## Ajax

允许浏览器与服务器通信而无需刷新当前页面的技术都叫做Ajax

Ajax并不是一个新的技术，而是多种技术的综合，包括Javascript、XHTML和CSS、DOM、XML和XMLHttpRequest服务器端语言：服务器需要具备向浏览器发送特定信息的能力。Ajax与服务器端语言无关

### ajax使用及步骤

1. 创建XMLHttpRequest对象，也就是创建一个异步调用对象
2. 创建一个新的HTTP请求，并指定该HTTP请求的方法、URL及验证消息
3. 设置响应HTTP请求状态变化的函数
4. 发送HTTP请求
5. 获取异步调用返回的数据
6. 使用JavaScript和DOM实现局部刷新

### XMLHttpRequest对象的三个常用的属性

1. onreadystatechange属性：存有处理服务器响应的函数
2. readyState:存有服务器响应的状态信息。每当readyState改变时，onreadystatechange函数就会被执行
    
    readyState属性可能的值：

    - 0：请求为初始化（调用open()之前）
    - 1：请求已提出（调用send()之前）
    - 2：请求已发送（在这里通常可以从响应得到内容头部）
    - 3：请求处理中（响应中通常有部分数据可用，但是服务器还没有完成响应）
    - 4：请求已完成（可以访问服务器响应并使用它）
3. responseText属性：取回服务器返回的数据

### xmlhttprequest方法

1. open()方法：open方法有三个参数。第一个参数定义发送请求所使用的方法，第二个参数规定服务器脚本的URL，第三个参数规定应当对请求进行异步处理

        xmlHttp.open("get","test.jsp",true)

2. send()方法：将请求送往服务器。

Get与Post

与POST相比，GET更简单也更快，并且大部分情况下都能用，但是在以下情况，使用POST请求：
1. 无法使用缓存文件（更新服务器上的文件或数据库）
2. 向服务器发送大量数据（POST没有数据量限制）
3. 发送包含未知字符的用户输入时，POST比GET更稳定也更可靠

### AJAX状态值与状态码区别

***AJAX状态值*** 是指，运行AJAX经历过的几种状态，无论访问是否成功都将响应的步骤，可以理解为AJAX运行步骤。如正在发送，正在响应等，有AJAX对象与服务器交互时所得；使用ajax.readState获得，由数字1~4组成

***Ajax状态码*** 是指，无论ajax访问是否成功，由HTTP协议根据所提交的信息，服务器所返回的HTTP头信息代码，该信息使用 **ajax.status** 所获得有200，404，500等

### AJAX的优势：

1. 通过异步模式，提升了用户体验
2. 优化了浏览器与服务器之间的传输，减少了不必要的数据往返，减少了带宽占用
3. Ajax引擎在客户端运行，承担了服务器的工作，减少了大用户量下的服务器负载

### AJAX的缺点：

1. ajax不支持浏览器back按钮
2. 安全问题AJAX暴露了与服务器交互的细节
3. 对搜索引擎的支持比较弱
4. 破坏了程序的异常机制
5. 不容易调试
