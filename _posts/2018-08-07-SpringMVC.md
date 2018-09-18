---
layout: post                  
title: "SpringMVC"             
date: 2018-08-07               
tag:  ssm框架
---

## SpringMVC

MVC设计模式：程序分层，分工合作，既相互独立，又协同工作

View-视图：解释模型，接收数据更新请求，发送用户输入给控制器，允许控制器选择视图

Controller控制器：接收用户请求，调用模型响应用户请求，选择视图显示相应结果

Model-模型：封装应用程序状态，响应状态查询，处理业务流程，通知视图业务状态更新

### MVC优点：

1. 多视图共享一个模型，提高了代码的可重用性
2. MVC三个模块相互独立，松耦合架构
3. 控制器提高了应用程序的灵活性和可配置性
4. 有利于软件工程化管理

### MVC缺点

1. 原理复杂
2. 增加了系统结构和实现的复杂性
3. 视图对模型数据的低效率访问

## 前端控制器

<p><img src="/images/Blog/lc.png" ></p>

用户发送请求到前端控制器，前端控制器将具体请求发送给控制器，控制器调用业务逻辑生成业务数据返回给前端控制器，前端控制器将业务数据发给业务视图，由业务视图呈现最终的用户视图，返回给前端控制器，由前端控制器发送给客户端。

前端控制器也称为调度器，将请求具体分发给控制器，由控制器进行业务数据的抽取

MVC核心思想：业务数据抽取同业务数据呈现相分离

SpringMVC：

- DispatcherServlet:是SpringMVC的前端控制器
    用户发送请求到DispatcherServlet,由DispatcherServlet分发给合适Controller，由Controller产生需要的业务数据Model，通过DispatcherServlet将Model传递给View完成最终的页面

- HandlerAdapter:Handler是DispatcherServlet内部的一个类，是controller的表现形式，通过@RequestMapping来识别Controller
    适配器模式，将各种不同类型的Handler适配成DispatcherServlet可使用的Handler

- HandlerInterceptor:拦截器  是一个接口，用在调用Controller之前，之后，  在Model发送到view的时候

- HandlerMapping :表示前端控制器与Controller之间映射关系，在            HandlerMapping工作完，可以给一个HandlerAdapter，包含Controller的实例

- ModelAndView ：Model的具体表现

- ViewResolver：视图解析器

<p><img src="/images/Blog/frontController.png" ></p>

request--->DispatcherServlet--->通过代理给HandlerMapping去找对应的Controller--->HandlerInterceptor--->返回HandlerAdapter给DispatcherServlet，HandlerAdapter被调用，得到Model，返回给视图解析器--->DispatcherServlet调用ViewResolver返回view，将Model传递给view

### SpringMVC拦截器与过滤器

Java里的拦截器是动态拦截Action调用的对象。它提供了一种机制可以使开发者定义一个在action执行前后执行的代码，也可以在一个action执行前阻止其执行

同时也提供了一种可以提取action可重用部分的方式。在AOP中拦截器用于在某个方法或字段被访问之前，进行拦截然后在之前或之后加入某些操作。

SpringMVC具有统一的入口DispatcherServlet,所有的请求都通过DispatcherServlet,所以只需要在DispatcherServlet上做文章即可

过滤器可以简单理解为"取你所需"，忽视掉那些不想要的东西
拦截器可以理解为"拒你所拒"，关心你想要拒绝哪些东西

1. 拦截器是基于java反射机制的，而过滤器是基于函数回调的
2. 过滤器依赖于servlet容器，而拦截器不依赖于servlet容器
3. 拦截器只对action起作用，而过滤器对所有请求起作用
4. 拦截器可以访问上下文，值栈里面的对象，而过滤器不能
5. 在action的生命周期中，拦截器可以多次调用，而过滤器只能在容器初始化的时候调用一次

### **拦截器的实现：**

- 编写拦截器类实现handInterceptor接口
- 将其注册到Spring MVC框架中
- 配置拦截器的拦截规则：
    1. preHandle：预处理的回调方法，实现处理器的预处理(登录检查),第三个参数为响应的处理器；返回值为true时表示继续流程，false表示是中断流程，不会继续调用其他拦截器或处理器，此时我们需要response来产生响应
    2. postHandle:后处理回调方法，实现处理器的后处理，但在渲染试图前，此时我们可以通过modelAndView对模型数据进行处理或对视图进行处理
    3. afterCompletion:整个请求处理完毕回调方法，即在视图渲染完毕时回调，如性能监控中我们可以记录结束时间并输出消耗时间，还可以进行一些资源清理，类似于try-catch-finally中的finally，但仅调用处理器执行链中preHandle返回true的拦截器的afterCompletion

