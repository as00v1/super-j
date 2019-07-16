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