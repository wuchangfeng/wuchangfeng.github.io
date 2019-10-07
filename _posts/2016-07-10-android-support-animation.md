---
title: 深入浅出 Android Support Annotation
toc: true
date: 2016-06-16 21:04:27
tags: android
categories:
description:
feature:
---

最近看一些大神的代码,发现了 Android 引入了几个很酷的注解类型,应该很早了，只是一直没用过。Google 了一下,转载了这篇文章。**下面这篇文章转自 [更上一层楼－Android研发工程师高级进阶](https://asce1885.gitbooks.io/android-rd-senior-advanced/content/shen_ru_qian_chu_android_support_annotations.html)**

##  一.  Nullness注解

使用@NonNull注解修饰的参数不能为null。在下面的代码例子中，我们有一个取值为null的name变量，它被作为参数传递给sayHello函数，而该函数要求这个参数是非null的String类型：

``` java
public class MainActivity extends ActionBarActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        String name = null;
        sayHello(name);
    }
    void sayHello(@NonNull String s) {
        Toast.makeText(this, "Hello " + s, Toast.LENGTH_LONG).show();
    }
}
```

如上,编译器自然会发出警告,按照前面的说法,道理也很简单。

## 二 . 资源类型注解

是否曾经传递了错误的资源整型值给函数，还能够愉快的得到本来想要的整型值吗？资源类型注解可以帮助我们准确实现这一点。在下面的代码中，我们的sayHello函数预期接受一个字符串类型的id，并使用@StringRes注解修饰：

``` java
public class MainActivity extends ActionBarActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        sayHello(R.style.AppTheme);
    }
    void sayHello(@StringRes int id) {
        Toast.makeText(this, "Hello " + getString(id), Toast.LENGTH_LONG).show();
    }

}
```

如上也是错误的,，因为按照注解,期待的也是字符串类型的符串资源 id。如下即可:

```java
sayHello(R.string.name);
```



## 三 .  IntDef和StringDef注解

很多时候，我们使用整型常量代替枚举类型（性能考虑），例如我们有一个IceCreamFlavourManager类，它具有三种模式的操作：VANILLA，CHOCOLATE和STRAWBERRY。我们可以定义一个名为@Flavour的新注解，并使用@IntDef指定它可以接受的值类型。

``` java
public class IceCreamFlavourManager {

    private int flavour;

    public static final int VANILLA = 0;
    public static final int CHOCOLATE = 1;
    public static final int STRAWBERRY = 2;

    @IntDef({VANILLA, CHOCOLATE, STRAWBERRY})
    public @interface Flavour {
    }

    @Flavour
    public int getFlavour() {
        return flavour;
    }

    public void setFlavour(@Flavour int flavour) {
        this.flavour = flavour;
    }

}
```

如前面所述,该注解也是限制类型,只不过限制的是给定的类型,因而 IDE 肯定会给出提示。