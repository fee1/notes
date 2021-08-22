# SpringMVC基础

## 1.SpringMVC的主要组件

### 1.前端请求控制器(DispatcherServlet)

- 作用：接受请求、响应结果，相当于转发器，有了DispatcherServlet就减少了其他组件之间的耦合度

### 2.处理器映射器(HandlerMapping)

- 根据请求的URL来查找Handler

### 3.处理器适配器(HandlerAdapter)

### 4.处理器Handler(需要程序员开发)

### 5.视图解析器(ViewResolver)

- 进行视图解析，根据视图逻辑名解析成真正的视图

### 6.视图View

- View是一个接口，它的实现类支持不同的视图类型(jsp，freemarker，pdf等等)

## 2.DispatcherServlet前端请求控制器

### Spring的MVC框架是围绕DispatcherServlet来设计的，它用来处理所有的HTTP请求和响应

### 问题

- 1.SpringMVC的控制器是不是单例模式，如果是，有什么问题，怎么解决？

	- 是单例模式，所以在多线程访问的时候有线程安全问题，不要用同步，会影响性能，解决方案是空hi其里面不能写公共的属性字段

- SpringMVC是怎么工作的？描述一下DispatcherServlet的工作流程？

	- 1.用户发送请求至前端控制器DispatcherServlet
	- 2.DispatcherServlet收到请求后，调用HandlerMapping处理器映射器，请求获取Handle
	- 3.处理器根据请求URL找到具体的处理器，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet
	- 4.DispatcherServlet调用HandlerAdapter处理器适配器
	- 5.HandlerAdapter经过适配调用具体处理器(Handler，也叫后端控制器)
	- 6.Handler执行完成返回ModelAndView
	- 7.HandlerAdapter将Handler执行结果ModelAndView返回给DispathcerServlet
	- 8.DispatcherServlet将ModelAndView传给ViewResolver视图解析器进行解析
	- 9.ViewResolver解析后返回具体View
	- 10.DispatcherServlet对View进行渲染视图(即将模型数据填充至视图中)
	- 11.DispatcherServlet响应用户

## 所有者权益

