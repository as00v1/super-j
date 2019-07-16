## J2EE概览

这部分主要对J2EE部分做一些系统总结，因为现在Spring全家桶称霸市场，把JavaWeb这部分包装地很彻底，所以哪怕没听说过Servlet都可以直接开发出接口，但是对于用心搞技术的人来讲，当然要本着打破砂锅问到底的精神，探其本质。

### **1. Servlet是什么？**
理论上的Servlet泛指JavaWeb中处理用户请求的程序，它负责接收用户HTTP请求，把用户请求封装成HttpServletRequest对象交给我们开发的程序处理，然后把我们需要返回的数据封装成HttpServletResponse对象，返回给请求端。