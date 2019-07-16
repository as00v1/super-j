## Java深入知识

这里面包含更深一点的Java知识，涉异常处理、序列化、反射操作等。

### **1. Java异常处理的结构？**
   
在 Java 中，所有的异常都有一个共同的祖先java.lang包中的 Throwable类。Throwable有两个重要的子类：
* **Exception**（异常）：指程序可以处理的异常情况，通常是由于开发人员的逻辑问题引起，通过try-catch块可以进行处理。常见的有：NullPointerException（空指针异常，访问的对象方法或属性时，对象没有指向任何实例）、ArrayIndexOutOfBoundsException（数组越界异常，数组下标超出了数组所在内存范围）
* **Error**（错误）：指程序不能处理的异常情况，错误一般与开发人员无关，往往是虚拟机也无法预料的异常，它没办法通过try-catch去处理。如OutOfMemoryError（内存溢出异常，指虚拟机无法再获取到程序所需的内存）、NoClassDefFoundError（无字节码文件异常，指虚拟机根据调用者的描述路径找不到响应的字节码）。这些异常发生时，Java虚拟机（JVM）一般会选择终止线程。

>异常的类图如图：  
![](images/exception.png)

*PS. 刚开始学习Java的时候，总觉得异常处理是个鸡肋的玩意，当时认为只要代码逻辑完善就永远不需要进行异常捕获和处理。后来实际工作中发现，总有异常是意想不到的发生，为了程序的稳定性、健壮性和体验友好，异常处理是非常重要的，希望初学者看到此处能够先树立好异常处理的观念，大佬请略过~*

### **2. Throwable类常用方法？**
 * `public string getMessage()`：返回异常发生时的详细信息，比较详细，一般记录进error级别的日志中
 * `public string toString()`：返回异常的简要信息
 * `public string getLocalizedMessage()`：返回异常对象的本地化信息。使用Throwable的子类覆盖这个方法，可以声称本地化信息。如果子类没有覆盖该方法，则该方法返回的信息与getMessage()返回的结果相同
 * `public void printStackTrace()`：在控制台上打印Throwable对象封装的异常信息

### **3. 异常处理的方式？**
   
一般使用try-catch-finally代码块来捕获和处理异常：
* `try`：try块内包含需要进行异常捕获的代码，它必须与catch或finally并存
* `catch`：用于对捕获的异常进行业务处理，通常是记录日志并返回友好的错误信息
* `finally`：无论什么情况，finally块中的代码总会被执行。除非代码块中出现异常，或者线程被终止、CPU关闭。

*PS.以下代码块，入参为2时返回值是什么？*
```java
public static int func(int value) {
    try {
        return value;
    } finally {
        if (value == 2) {
            return 0;
        }
    }
}
```
答案是0，因为真正返回之前会执行finally内的逻辑，并且覆盖原值。

### **4. 序列化和反序列化是什么？如何操作？**

* **序列化**：将对象转换为字节序列的过程。

Java提供了java.io.ObjectOutputStream类来完成对对象的序列化操作，要序列化的对象的类必须实现java.io.Serializable接口，否则会抛出异常。序列化代码如下如下：
```java
private static void ser(Object o) {
    ObjectOutputStream os = null;
    try {
        os = new ObjectOutputStream(new FileOutputStream(new File("1.txt")));
        os.writeObject(o);
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (os != null) {
            try {
                os.close();
            } catch (IOException e) {}
        }
    }
}
```
*PS. 这里使用了一个文件保存了序列化后的内容，其实序列化不仅仅可以保存到磁盘文件里，也可以保存到数组、缓冲区内，都可以通过ObjectOutputStream的构造方法实现，感兴趣的话可以试试。  
另外，我个人理解的序列化也不仅仅是把对象转化为二进制，广义上来说，任何可以**把对象转化为某个序列进行持久存储**的方式我认为都是一种序列化，比如把对象转化为Json格式字符串存储。*

* **反序列化**：将字节序列转化为对象的过程。  
Java提供了java.io.ObjectInputStream类来完成对对象的反序列化操作，如下：
```java
private static Object deser() {
    Object o = null;
    ObjectInputStream os = null;
    try {
        os = new ObjectInputStream(new FileInputStream(new File("1.txt")));
        o = os.readObject();
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    } finally {
        if (os != null) {
            try {
                os.close();
            } catch (IOException e) {}
        }
    }
    return o;
}
```
可以理解反序列化就是按照序列化时的写入方式再把对象还原的过程。

### **5. 如果有些字段不想进行序列化，怎么办？**
Java提供了`transient`关键字，可以声明此属性不被序列化。

### **6. 什么是反射？**
>在计算机科学中，反射是指计算机程序在运行时（Run time）可以访问、检测和修改它本身状态或行为的一种能力。用比喻来说，反射就是程序在运行的时候能够“观察”并且修改自己的行为。  
在面向对象的编程语言如Java中，反射允许在编译期间不知道接口的名称，字段（fields，即成员变量）、方法的情况下，在运行时检查类、接口、字段和方法。它还允许根据判断结果进行实例化新对象和不同方法的调用。  
*--摘抄自[维基百科](https://zh.wikipedia.org/wiki/%E5%8F%8D%E5%B0%84_(%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6))*

用通俗的话讲，在Java中反射我理解是指一个对象在编译时并不确定对象是否生成，而是在运行期动态生成对象的一种技术。反射的意义在于在程序运行时可以加载和运行任意可以通过编译时运行的类或方法。

### **7. Java中反射有哪些玩法？**
这个问题在面试中一般就是考察点实际的东西，因为反射这个东西理论太枯燥了，如果能说出来点实际的东西证明你会玩，那我觉得反射这块已经没什么可以再深究的了。  
现在我们先简单创建一个类，包含一个构造方法和两个属性：
```java
public class Person {

	private String name;
	private int age;
	
	public Person(String name, int age) {
		this.name = name;
		this.age = age;
	}
	
	public void print() {
		System.out.println("name = " + name + ", age = " + age);
	}
}
```
然后，创建一个测试类：
```java
public class Test {

	public static void main(String[] args) {
		try {
			// 获取类信息
			Class<Person> clazz = Person.class;
			// 通过类的全限定名获取类信息
//			Class<Person> clazz = (Class<Person>) Class.forName("com.test.reflect.Person");
			System.out.println("类的名称：");
			System.out.println(clazz.getName());
			System.out.println("类的属性列表：");
			Field[] fields = clazz.getFields();
			for (Field field : fields) {
				System.out.println(field.getType().getName() + " " + field.getName());
			}
			System.out.println("类的方法列表：");
			Method[] methods = clazz.getMethods();
			for (Method method : methods) {
				// 方法的返回值类型  和  方法名称
				System.out.print(method.getReturnType().getName() + " " + method.getName() + "(");
				// 方法的参数类型
				Class<?>[] parameterTypes = method.getParameterTypes();
				for (Class<?> parameterType : parameterTypes) {
					System.out.print(" " + parameterType.getName());
				}
				System.out.println(")");
			}
			System.out.println("构造方法：");
			Constructor<?>[] cons = clazz.getConstructors();
			for (Constructor<?> constructor : cons) {
				// 构造方法名称
				System.out.print(constructor.getName() + "(");
				// 构造方法的参数类型
				Class<?>[] parameterTypes = constructor.getParameterTypes();
				for (Class<?> parameterType : parameterTypes) {
					System.out.print(" " + parameterType.getName());
				}
				System.out.println(")");
			}
			
			System.out.println("创建实例：");
			// 使用第1个构造方法实例化
			Person person = (Person)cons[0].newInstance("张三", 1);
			person.print();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
```

如果不想看代码，记住以下几个关键类：

* **java.lang.Class**：关键类，封装了**类**的信息和各种操作类的方法，如类的全限定名、获取类的属性列表、获取方法和构造方法列表等等
* **java.lang.reflect.Field**：封装了**属性**的信息，可以获取属性的名称、类型等
* **java.lang.reflect.Method**：封装了**方法**的信息，可以通过此类型的实例获取方法名、返回值、入参格式、异常类型等
* **java.lang.reflect.Constructor**：封装了**构造方法**的信息，可以获取构造方法的参数类型等，并且可以通过此类的实例调用`newInstance`方法创建类的实例