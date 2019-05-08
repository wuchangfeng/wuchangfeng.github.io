---
title: 利用 APT 在 Java 文件编译时获取注解信息
toc: true
date: 2018-05-13 19:01:45
tags: java
categories: About Java
description:
feature:
---
运行时利用反射获取 Annotation 信息，当反射过多，自然效率不高。自然有另外的办法解决，即在编译时获取注解信息。相当于直接调用。

根据 Crazy Java 中的例子类学习一下，如何利用 APT 在编译时获取注解信息。


<!--more-->

### 一 . 概念理解

**编译**时处理**注解**：APT 一种注解处理工具，对源文件进行检测，找出源文件包含的注解信息，然后针对注解信息做处理。

而 Hibernate 框架就具备该功能，可以在 java 源文件中放置一些 Annotation，然后使用 APT 工具根据该 Annotation 生成一份 XML 文件。



Java 提供的 javac.exe 工具有一个 -processor 选项，他可以指定一个 Annotation 处理器，在编译源文件时通过其指定了 Annotation 处理器，那么这个 Annotation 处理器将会在编译时提取并处理 java 源文件中的 Annotation。

Annotation 处理器需要实现 javax.annotation.process 包下的 Processor 接口，并且实现该接口中的所有方法，会采用继承 AbstractProcessor 方式来实现 Annotation 处理器。

### 二 . 实例代码

**注解文件**

@Retention(RetentionPolicy.SOURCE) 是编译是注解的特性，标志。

用于修饰成员变量(FIELD),另外注意在注解定义中成员变量后面要加 ()，该注解作用用来持久化类

``` java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.SOURCE)
@Documented
public @interface Id
{
	String column();
	String type();
	String generator();
}
```

用来修饰类或者接口

``` java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.SOURCE)
@Documented
public @interface Persistent
{
	String table();
}
```
用来修饰成员变量

``` java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.SOURCE)
@Documented
public @interface Property
{
	String column();
	String type();
}
```

**被注解修饰的文件**

``` java
@Persistent(table="person_inf")
public class Person
{
	@Id(column="person_id",type="integer",generator="identity")
	private int id;
	@Property(column="person_name",type="string")
	private String name;
	@Property(column="person_age",type="integer")
	private int age;

	// constrcut
	public Person()
	{
	}
	// constrcut
	public Person(int id , String name , int age)
	{
		this.id = id;
		this.name = name;
		this.age = age;
	}
 
	// getter and setter  
}
```

**ADT工具类**

``` java
@SupportedSourceVersion(SourceVersion.RELEASE_7)
// 指定可以处理这三个 Annotation
@SupportedAnnotationTypes({"Persistent" , "Id" , "Property"})
public class HibernateAnnotationProcessor
	extends AbstractProcessor
{
	// 循环处理每个需要处理的程序对象
	public boolean process(Set<? extends TypeElement> annotations
		, RoundEnvironment roundEnv)
	{
		// 定义文件输出流，用于生成文件
		PrintStream ps = null;
		try
		{
			// 遍历每个被 @Persistent 修饰的 class 文件
			for (Element t : roundEnv.getElementsAnnotatedWith(Persistent.class))
			{
				// 获得正在处理的类名
				Name clazzName = t.getSimpleName();
				// 获取类定义前的 @Persistent Annotation
				Persistent per = t.getAnnotation(Persistent.class);
				
				ps = new PrintStream(new FileOutputStream(clazzName
					+ ".hbm.xml"));
				
				ps.println("<?xml version=\"1.0\"?>");
				ps.println("<!DOCTYPE hibernate-mapping PUBLIC");
				ps.println("	\"-//Hibernate/Hibernate "
					+ "Mapping DTD 3.0//EN\"");
				ps.println("	\"http://www.hibernate.org/dtd/"
					+ "hibernate-mapping-3.0.dtd\">");
				ps.println("<hibernate-mapping>");
				ps.print("	<class name=\"" + t);
				ps.println("\" table=\"" + per.table() + "\">");
				for (Element f : t.getEnclosedElements())
				{
					// 只处理成员变量上的 annotation
					if (f.getKind() == ElementKind.FIELD)   
					{
						// 获取成员变量定义前的 Annotation
						Id id = f.getAnnotation(Id.class);      
						// 当 @Id Annotation 存在时输出 <id.../>元素
						if(id != null)
						{
							ps.println("		<id name=\""
								+ f.getSimpleName()
								+ "\" column=\"" + id.column()
								+ "\" type=\"" + id.type()
								+ "\">");
							ps.println("		<generator class=\""
								+ id.generator() + "\"/>");
							ps.println("		</id>");
						}
						// 获取成员变量定义前的@Property Annotation
						Property p = f.getAnnotation(Property.class);  
						// 输出
						if (p != null)
						{
							ps.println("		<property name=\""
								+ f.getSimpleName()
								+ "\" column=\"" + p.column()
								+ "\" type=\"" + p.type()
								+ "\"/>");
						}
					}
				}
				ps.println("	</class>");
				ps.println("</hibernate-mapping>");
			}
		}
		catch (Exception ex)
		{
			ex.printStackTrace();
		}
		finally
		{
		 // close
		}
		return true;
	}
}
```

提供了 Annotation 处理器之后，就可以使用带 -processor 选项的 javac.exe 命令来编译 Person.java 了

``` java
	javac -processor HibernateAnnotationProcessor Person.java
```

生成的 XML 文件如下：

``` xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC
	"-//Hibernate/Hibernate Mapping DTD 3.0//EN"
	"http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
	<class name="Person" table="person_inf">
		<id name="id" column="person_id" type="integer">
		<generator class="identity"/>
		</id>
		<property name="name" column="person_name" type="string"/>
		<property name="age" column="person_age" type="integer"/>
	</class>
</hibernate-mapping>
```


XML 文件中浅绿色字体就是 Person.java 中的 Annotation 部分。从这份生成的 XML 文件中可以看出 APT 工具确实可以简化程序的开发，可以将一些关键信息通过 Annotation 写在程序中，然后利用 APT 工具生成额外的文件。



### 三 . 分析总结

* 与通过反射来获取信息不同的是，此处 Annotation 处理器使用 RoundEnvironment 来获取注解信息，RoundEnvironment 包含了一个 getElementsAnnotatedWith() 方法，可以根据 Annotation 获取需要处理的程序单元。
* Element 里包含一个 getKind()，其返回 Element 所代表的程序单元，返回值可以为 ElementKind.CLASS(类)，ElementKind.FIELD(成员变量)...
* Element 还包含了 getEnclosedElement() 方法，可获取该 Element 里面定义的所有程序单元，包括成员变量，方法和构造器等等。
* 接下来程序只处理成员变量前面的 Annotation,然后程序调用了 Element提供的 getAnnotation() 来获取修饰该 Element 的 Annotation。