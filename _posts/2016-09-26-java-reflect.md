---
title: 深入理解 Java 之反射机制的应用
toc: true
date: 2016-05-17 21:27:37
tags: reflect
categories: About Java
description:
feature:
---

本篇文章介绍反射、利用反射获得 Class 对象所对应类的基本信息以及利用 Class 对象创建对应类的实例并操作。

## 引入

Java 许多对象在运行时候都会出现两种类型：编译时类型和运行时类型，如下代码：

``` java
Person p = new Student();
```

这行代码会产生一个 p 变量，它在编译时类型为 Person，但是在运行时候类型为 Student。为了解决类型不一致的问题，程序需要在运行时候知道对象的真实信息。有以下两种办法：

* 假设在编译时和运行时完全知道类型的具体信息。可以先使用 instanceof 运算符进行判断，再利用强制类型转换成运行时类型即可。
* 在编译时无法预知对象和类可能属于哪些类，程序只能在运行时来发现其具体信息，这时候就要用到反射了。

## 零.获得 Class 对象

每个类被加载之后，系统会对该类生成一个 Class 对象，通过该对象就可以访问 JVM 中对应的类。有以下三种方式可以获得 Class 对象：

* 使用 Class 类的 forName() 方法
* **调用类的 class 属性来获得 Class 对象**
* 调用某个类的 getClass 方法

主要可以通过 Class 对象获得类的信息主要如下：

* 获得 Class 对象所对应类所包含的构造器
* 获得 Class 对象所对应类所包含的方法
* 获得 Class 对象所对应类所包含的注解
* 获得 Class 对象所对应类所包含的内部类和所继承的父类

实例为获得 Class 对象所对应类的全部 public 方法：

```java
Class<Test> clazz = Test.class;
Method[] mtds = clazz.getMethods();
for(Method md : mtds){
  //... md 
}
```

## 一 . 利用 Class 对象创建对应实例

**不能通过具体的方法来获得类对象，所以我们采用反射的方式来**这句话是错误的，前面的方法只是**获取 class 对象，不是实例**，我们基于前面获得 class 对象，然后根据反射来获取相应的实例。

1. 一种是利用 Class 对象的 newInstance() 方法来创建该 Class 对象对应类的实例。要求有默认构造器。
2. 另一种先使用 Class 对象获取指定的 Construct 对象，再调用 Construct 对象的 newInstance() 方法来创建。

第一种方法比较常见，因为很多 javaEE 框架都要求根据配置文件信息来创建 java 对象，**从配置文件中只是读取某个类的字符串名字**，程序需要根据该字符串来创建对应的实例，就必须要使用反射。

```java
public class ObjectPoolFactoryTest {
    // 对象池
    private Map<String,Object> objectpool = new HashMap<>();

    // 定义一个创建对象的方法
    private Object createObject(String clazzName) throws
            InstantiationException
        ,IllegalAccessException,ClassNotFoundException{

        Class<?> clazz = Class.forName(clazzName);

        return clazz.newInstance();
    }
    // 根据指定文件来初始化对象池
    public void initPool(String fileName)
        throws InstantiationException
        ,IllegalAccessException,ClassNotFoundException{
        try {
            
            FileInputStream fis = new FileInputStream(fileName);
            Properties prop = new Properties();
            prop.load(fis);
            // 取出一对key-value，就根据value创建一个对象，并添加到对象池中
            for (String name : prop.stringPropertyNames()){

                objectpool.put(name,createObject(prop.getProperty(name)));
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public Object getObject(String name){

        return objectpool.get(name);
    }
    
    public static void main(String[] args) throws Exception{

        ObjectPoolFactoryTest pf = new ObjectPoolFactoryTest();
        pf.initPool("G:\\JavaTest\\src\\ttt.txt");
        System.out.println(pf.getObject("a"));
        System.out.println(pf.getObject("b"));
   		 }
}
```

这种使用配置文件来配置对象，然后由程序根据配置文件来创建对象的方式非常有用, Spring 框架就是采用这种方式。具体的其采用的是 XML 文件。



## 二 . 调用方法

当获得某个类的对应的 Class 对象之后，即可以通过该 Class 对象的 getMethods() 方法或者 getMethod() 来获取全部或者指定方法，返回 Method 数组或者 Method 对象，**每个 Method **对象对应一个方法，获得该对象后，程序就可以利用 Method 来获取其对应的方法，在 Method 里包含一个 invoke() 方法，其签名如下：

```java
Object invoke(Object obj,Object...args)
```

其中 obj 是执行该方法的**主调**，后面的 args 是执行该方法时传入该方法的实参。

一个例子的核心方法如下所示：

```java
// 该方法根据指定文件来初始化对象池，
// 它会根据配置文件来创建对象
public void initProperty()throws InvocationTargetException
		,IllegalAccessException,NoSuchMethodException
	{
		for (String name : config.stringPropertyNames())
		{
			// 每取出一对key-value对，如果key中包含百分号（%）
			// 即可认为该key是用于为对象的Field设置值，
			// %前半为对象名字，后半为Field名
			// 程序将调用对应的setter方法来为对应Field设置值。
			if (name.contains("%"))
			{
				// 将配置文件中key按%分割
				String[] objAndProp = name.split("%");
				// 取出需要设置Field值的目标对象
				Object target = getObject(objAndProp[0]);
				// 该Field对应的setter方法名:set + "属性的首字母大写" + 剩下部分
				String mtdName = "set" + 
				objAndProp[1].substring(0 , 1).toUpperCase() 
					+ objAndProp[1].substring(1);
				// 通过target的getClass()获取它实现类所对应的Class对象
				Class<?> targetClass = target.getClass();
				// 获取该属性对应的setter方法
				Method mtd = targetClass.getMethod(mtdName , String.class);
				// 通过Method的invoke方法执行setter方法，
				// 将config.getProperty(name)的属性值作为调用setter的方法的实参
				mtd.invoke(target , config.getProperty(name));
			} 
		}
}
```

Spring 框架就是通过上面这种方式将成员变量值以及依赖对象放置在配置文件中进行管理的，从而实现了较好的解耦。即 Spring 框架 IoC 的秘密。

## 三. 访问成员变量值

使用 Class 对象的 getFields() 或者 getField() 就可以获取该类所包括的全部成员变量或者指定成员变量。 

```java
class Student{
    // 私有的，外部没有方法来改变
    private String name;
    private int age;

    @Override
    public String toString() {
        return "Student[name:" + name + ",age:" + age + "]";
   	 }
	}

	public class FiledTest {

    public static void main(String[] args) throws Exception{

        Student s = new Student();
        System.out.println(s);
        // lei.class 来获取 class 对象
        Class<Student> studentClass = Student.class;
        Field nameField = studentClass.getDeclaredField("name");
        // 使该成员变量可以被修改
        nameField.setAccessible(true);
        nameField.set(s,"Yeek");
        // 可以获取所有的成员变量方法
        Field ageField = studentClass.getDeclaredField("age");
        ageField.setAccessible(true);
        ageField.set(s,19);
        System.out.println(s);
    	}
}
```

可以看见本来**name和age属性都是私有的**，外部程序是不能来改变的，但是**nameField.setAccessible(true)** 这一句却成功的让改变发生了。

所以着重看一下： setAccessible(boolean flag):将 Method 对象的 accessible 设置为指定的布尔值。为 true 时，指示该 method 在使用时应该取消 Java 语言的访问权限。

其属于父类 AccessibleObject，因此 Method，Construct，Field 都可以调用该方法，从而通过**反射**来调用 private 成员变量，构造器，方法。

## 四 . 操作数组

java.lang.reflect 中提供了一个 Array 类，Array 对象可以代表所有的数组，程序可以通过使用 Array 来**动态的创建数组，操作数组等等**

```java
public class ArrayTest1
{
	public static void main(String args[])
	{
		try
		{
			// 创建一个元素类型为String ，长度为10的数组
			Object arr = Array.newInstance(String.class, 10);
			// 依次为arr数组中index为5、6的元素赋值
			Array.set(arr, 5, "疯狂Java讲义");
			Array.set(arr, 6, "轻量级Java EE企业应用实战");
			// 依次取出arr数组中index为5、6的元素的值
			Object book1 = Array.get(arr , 5);
			Object book2 = Array.get(arr , 6);
			// 输出arr数组中index为5、6的元素
			System.out.println(book1);
			System.out.println(book2);
		}
		catch (Throwable e)
		{
			System.err.println(e);
		}
	}
}
```

## 五 . 反射机制的优点与缺点

反射机制的优点与缺点
为什么要用反射机制？直接创建对象不就可以了吗，这就涉及到了动态与静态的概念

静态编译：在编译时确定类型，绑定对象,即通过。
动态编译：运行时确定类型，绑定对象。动态编译最大限度发挥了java的灵活性，体现了多态的应用，有以降低类之间的藕合性。

**优点**

可以实现动态创建对象和编译，体现出很大的灵活性，特别是在J2EE的开发中它的灵活性就表现的十分明显。比如，一个大型的软件，不可能一次就把把它设计的很完美，当这个程序编译后，发布了，当发现需要更新某些功能时，我们不可能要用户把以前的卸载，再重新安装新的版本，假如这样的话，这个软件肯定是没有多少人用的。采用静态的话，需要把整个程序重新编译一次才可以实现功能的更新，而采用反射机制的话，它就可以不用卸载，只需要在运行时才动态的创建和编译，就可以实现该功能。

**缺点**

对性能有影响。使用反射基本上是一种解释操作，我们可以告诉JVM，我们希望做什么并且它满足我们的要求。这类操作总是慢于只直接执行相同的操作。