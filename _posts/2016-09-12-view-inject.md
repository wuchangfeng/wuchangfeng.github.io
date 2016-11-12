---
title: 利用 Annotation 实现基本的 View Inject 
date: 2016-05-13 15:51:05
tags: android
categories: About Java
---

用注解以及反射实现 Android 中的 injectView。

<!--more-->

Spring 的注解是使用 Java 反射机制实现的，当然如果让我们实现注解的话，可能往往也是想到利用反射来实现。但是我们知道如果通过反射，是在运行时（Runtime）来处理 View 的绑定等一些列事件的，这样比较耗费资源，会影响应用的性能。所以 ButterKnife 利用的是上文中提到的Java Annotation Processor 技术，自定义了我们平时常用的一些注解，并注册相应的注解处理器，最后生成了相应的辅助类。在编译时直接通过辅助类来完成操作，这样就不会产生过多的消耗，而出现性能问题。

这里先用反射来实现 Inject View ，初步来学习一下注解以及反射的实战应用。

另外对于原文的 EventInject 表示不太理解，不知道有没有更好的实现方式。

### 一 . 替代 SetContentView()

在 activity 中加载布局，如果采用注解的话，形式如下：

```java
@ContentView(R.layout.activity_main)
public class MainActivity extends AppCompatActivity {
```

**定义注解文件**：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface ContentView {

    int value();// for layout 
}
```

1. @Target(ElementType.TYPE) 用来修饰 MainActivity 类文件，其还可以指定用来修饰接口
2. @Retention(RetentionPolicy.RUNTIME) 用来表示 JVM 运行时也不会丢弃它，保证了可以利用反射来获取相关注解信息
3. @ interface 就是用来声明注解的，不要与接口搞混

```java
public static void injectContentView(Activity activity){
	
	Class<? extends Activity> clazz = activity.getClass();
        ContentView contentView = clazz.getAnnotation(ContentView.class);
        if (contentView != null) {
          
            int layoutId = contentView.value();
            try {
                Method setViewMethod = clazz.getMethod("setContentView", int.class);
                setViewMethod.invoke(activity, layoutId);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
}
```

1. activity.getClass() 调用 activity 对象的 getClass() 文件来获取 Class 对象(Class 对象在每个类被加载之后就会生成，而通过 Class 对象，在 JVM 中就可以访问到对应的类，即运行时)
2. clazz.getAnnotation(ContentView.class) 从 Class 中获取对应的注解信息
3. clazz.getMethod("setContentView", int.class) 获取 clazz 对应类的指定 setContent方法
4. setViewMethod.invoke(activity, layoutId) 该方法中 activity 是 invoke() 的调用者，而 layoutId 为执行该方法传入的实参 

**invoke()** 在动态代理中也有提及，可以搞一个实例来说明一下其用法：

Hello 类

```java
public class Hello {

　　public void foo(String name) {
　　　　System.out.println("Hello, " + name);
　　}
}
```

编写另外一个类来反射调用 Hello 上的方法： 

```java
import java.lang.reflect.Method;

public class Test{

　　public static void main(String[] args) throws Exception {
　　　　Class<?> clz = Class.forName("Hello");
　　　　Object o = clz.newInstance();
　　　　Method m = clz.getMethod("foo", String.class);
　　　　for (int i = 0; i < 16; i++) {
　　　　　　m.invoke(o, Integer.toString(i));
　　　　}
　　}
}
```

整个流程即 对象 o 来调用 Hello 类中的 foo() 方法，并且以此传入 Integer.toString(i) 参数。注意 Test 类在执行 main 之前，并没有持有 Hello 类的依赖，而是对 A 做运行时动态加载。

而关于 method.invoke() 方法的深入研究可以参考 [JAVA深入研究——Method的Invoke方法。](http://www.cnblogs.com/onlywujun/p/3519037.html)

### 三 . 替代 FindViewById()

在 MainActivity 中使用

```java
@ViewInject(R.id.text_view)
private TextView textView;
```

定义注解文件

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ViewInject {
    int value();
}
```

injectView 的实现

```java
private static void injectView(Activity activity) {
        Class<? extends Activity> clazz = activity.getClass();
        // get all fileds from activity
        Field[] fields = clazz.getDeclaredFields();
        for (Field field : fields) {
            // get all annnotations accrdoing to field
            ViewInject viewInject = field.getAnnotation(ViewInject.class);
            if (viewInject != null) {
                int viewId = viewInject.value();
                View view = activity.findViewById(viewId);
                try {
                    field.setAccessible(true);
					// here is the point
                    field.set(activity, view);

                } catch (IllegalAccessException e) {
                    e.printStackTrace();
                }
            }
        }
}
```

利用注解来实现 ViewInject 还是比较简单的，相对于 onCLick() 的替代方案

### 三 . 替代 OnClick()

在 MainActivity 中使用

```java
@OnClick(R.id.text_view)
    private void onClick(View view){
        textView.setText("hello allen");
    }
```

定义注解文件

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface OnClick {

    int [] value();
}
```

injectEvent 的实现

```java
 private static void injectEvent(final Activity activity) {

        Class<? extends Activity> clazz = activity.getClass();
        Method[] methods = clazz.getDeclaredMethods();
        for (final Method method2 : methods) {
            OnClick click = method2.getAnnotation(OnClick.class);
            if (click != null) {

                int[] viewId = click.value();
                method2.setAccessible(true);
                Object listener = Proxy.newProxyInstance(View.OnClickListener.class.getClassLoader(),
                        new Class[]{View.OnClickListener.class}, new InvocationHandler() {
                            @Override
                            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                                return method2.invoke(activity, args);
                            }
                        });

                try {
                    for (int id : viewId) {
                        View v = activity.findViewById(id);
                        Method setClickListener = v.getClass().getMethod("setOnClickListener", View.OnClickListener.class);
                        setClickListener.invoke(v, listener);
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }
```



OnClick 事件采用的是动态代理的思想。其中核心的应该还是 method2.invoke(activity, args),而之所以采取动态代理是因为 View.OnClickListener是一个接口，不能用反射来获得他的实例。

在 Crazy Java 中反射也说明了这一点，只能判断其是否为 Interface 而不能获取其实例。

### 四 . 参考

[hongyang的文章值得看看](http://blog.csdn.net/lmj623565791/article/details/39275847)

[ButterKnife 的源码分析](http://www.jianshu.com/p/0f3f4f7ca505)

[java基础之注解(annotation)](http://www.jianshu.com/p/ca7f22b4b751)