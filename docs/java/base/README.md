# Java基础知识

## Java理论
1. 什么是Java？
   
    Java是一门面向对象编程语言，具有面向对象特性（封装、继承、多态），通过JVM实现了跨平台的能力，并对指针操作进行了封装，因此使用起来更加简单安全；对多线程和网络编程有更好的支持。

2. JVM、JDK、JRE是什么？
   
   **JVM**

   Java Virtual Machine，Java虚拟机是能够运行Java字节码文件（.class文件）的虚拟机，它针对不同的系统有不同的实现，效果就是对于同一个字节码文件，在不同的系统都有相同的结果，这也是Java 语言“一次编译，随处可以运行”的关键。

   另外需要注意的是，JVM并不是针对Java语言的，而是针对字节码文件的，因此只要编译器能生成规范的字节码文件，理论上JVM也能够运行。目前流行的主要有：Groovy、Kotlin、Scala等。

   **JRE**

   Java Runtime Environment，Java运行环境。顾名思义，它包含了运行一个Java程序所必须的环境，如JVM、Java类库、Java运行命令和其他一些基础组件。我理解，它实现的是字节码文件能够正常地运行。

    **JDK**

    Java Development Kit，Java开发工具包，包含了：

    * JRE
    * Java编译器（javac）
    * 工具（javadoc和jdb）
    
    可见，JDK不仅能实现运行字节码文件的功能，还实现了Java文件编译成字节码文件的功能，是开发Java程序必须的环境。