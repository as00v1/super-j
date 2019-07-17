## J2EE概览

这部分主要对J2EE部分做一些系统总结，因为现在Spring全家桶称霸市场，把JavaWeb这部分包装地很彻底，所以哪怕没听说过Servlet都可以直接开发出接口，但是对于用心搞技术的人来讲，当然要本着打破砂锅问到底的精神，探其本质。

### **1. Servlet是什么？**
理论上的Servlet泛指JavaWeb中处理用户请求的程序，它负责接收用户HTTP请求，把用户请求封装成HttpServletRequest对象交给我们开发的程序处理，然后把我们需要返回的数据封装成HttpServletResponse对象，返回给请求端。

### **2. Servlet接口有哪些方法？**
* `void init(ServletConfig var1) throws ServletException`：初始化方法，用于Servlet创建时执行一次

* `ServletConfig getServletConfig()`：获取Servlet配置对象

* `void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException`：用来处理用户请求

* `String getServletInfo()`：获取Servlet信息

* `void destroy()`：容器销毁前执行，销毁方法

### **3. Servlet的生命周期？**
1. Web容器加载Servlet并将其实例化后，Servlet生命周期开始；
2. 容器运行其`init()`方法进行Servlet的初始化；
3. 请求到达时，调用`service()`方法处理用户请求；
4. 容器关闭时，执行`destroy()`方法进行资源的关闭处理。

一个Servlet的生命周期大致如上所述，其中`init()`和`destroy()`方法分别只会在初始化时和销毁时执行一次，`service()`方法每次请求到达时都会执行。

### **4. Servlet是线程安全的吗？**
不是。  
Servlet在web容器启动时就完成了实例化，在整个web应用的生命周期中，Servlet实例是以单例的模式存在，可以节省资源，但是线程不安全，因此需要避免在Servlet中定义属性。

### **5. JSP是什么？**
Java Server Pages简称JSP，看名字理解，它是一种运行在服务器上的页面，使用Java语言创建。与Servlet类似，它也能够处理用户请求，不同的是，Servlet的应用逻辑是在Java文件中，并且完全从表示层中的HTML分离开来，而JSP的情况是Java和HTML组合成一个扩展名为.jsp的文件。因此在MVC设计模式中，JSP更多的是作为view层，而Servlet是作为Controller层存在。

### **6. JSP内置对象有哪些？**
JSP有9个内置对象：

* request：封装客户端的请求，其中包含来自GET或POST请求的参数；
* response：封装服务器对客户端的响应；
* pageContext：通过该对象可以获取其他对象；
* session：封装用户会话的对象；
* application：封装服务器运行环境的对象；
* out：输出服务器响应的输出流对象；
* config：Web应用的配置对象；
* page：JSP页面本身（相当于Java程序中的this）；
* exception：封装页面抛出异常的对象。

### **7. Request对象的主要方法有哪些？**
这里记录了servlet封装好的request对象的方法，个人感觉记几个主要的即可。
- setAttribute(String name,Object)：设置名字为name的request 的参数值 
- getAttribute(String name)：返回由name指定的属性值 
- getAttributeNames()：返回request 对象所有属性的名字集合，结果是一个枚举的实例 
- getCookies()：返回客户端的所有 Cookie 对象，结果是一个Cookie 数组 
- getCharacterEncoding() ：返回请求中的字符编码方式 = getContentLength() ：返回请求的 Body的长度 
- getHeader(String name) ：获得HTTP协议定义的文件头信息 
- getHeaders(String name) ：返回指定名字的request Header 的所有值，结果是一个枚举的实例 
- getHeaderNames() ：返回所以request Header 的名字，结果是一个枚举的实例 
- getInputStream() ：返回请求的输入流，用于获得请求中的数据 
- getMethod() ：获得客户端向服务器端传送数据的方法 
- getParameter(String name) ：获得客户端传送给服务器端的有 name指定的参数值 
- getParameterNames() ：获得客户端传送给服务器端的所有参数的名字，结果是一个枚举的实例 
- getParameterValues(String name)：获得有name指定的参数的所有值 
- getProtocol()：获取客户端向服务器端传送数据所依据的协议名称 
- getQueryString() ：获得查询字符串 
- getRequestURI() ：获取发出请求字符串的客户端地址 
- getRemoteAddr()：获取客户端的 IP 地址 
- getRemoteHost() ：获取客户端的名字 
- getSession([Boolean create]) ：返回和请求相关 Session 
- getServerName() ：获取服务器的名字 
- getServletPath()：获取客户端所请求的脚本文件的路径 
- getServerPort()：获取服务器的端口号 
- removeAttribute(String name)：删除请求中的一个属性 

### **8. 转发(Forward)和重定向(Redirect)的区别？**
- **转发**：转发是服务器端行为，相当于服务器将用户请求的request和response移交给另一个处理器去处理，对于客户端来说可以是无感知的
- **重定向**：重定向是客户端（浏览器）行为，服务端会把第一次请求的response.status状态码设置为301或者302，告诉客户端资源已经转移到了新地址，需要重新发起请求。  

总结来说：
- 从地址栏来看：转发地址栏不变，重定向会变
- 从数据共享来看：转发在服务器端是共享数据的，重定向不会
- 从请求次数来看：转发是一次请求，重定向向服务器请求了两次
- 从效率来看：转发效率高，重定向效率低