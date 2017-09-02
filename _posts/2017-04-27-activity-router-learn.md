---
layout: post
title: 深入理解 Android 之路由框架源码解析
date: 2017-01-18 16:54:22 +0800
categories: android
---

ActivityRouter 源码分析

感觉 Android 中的这个路由机制还是比较有意思的，研究了一段时间。写篇文章记录一下。个人认为能够带来的一些好处如下：

- 炫酷。注解的特性，路由机制听着就很不错。好吧，其实相对于传统的跳转机制是简洁。
- 高度解耦。好吧，其实 Intent 机制本身就是能够实现解耦功能。
- 组件化开发。提前配置具有关系的页面映射，具备协同工作机制。
- 备灾功能。如果原先预备跳转的界面不存在，可以跳转至一个临时的备灾界面。或者说拦截功能。

主要的核心知识点如下：

- Android 中的 Intent 机制
- Java 反射机制
- Android 的编译时注解
- APT 技术以及 Javapoet 技术

在进行源码分析之前，先总结一下其大概的原理，就是用一个 Map 来存储具体的路由、实体 Activity 之间的映射。然后在利用 Android 自身的 Intent 机制，来进行界面的跳转。

> 简单使用过程

根据 readme 来说明一个简单的使用过程：

#### 全局注册

注意注入的这个字段一定要是全局唯一的，避免与其他模块冲突：

```java
Router.init("test");//设置Scheme
```

#### 定义路由

给 SecondActivity 定义了路由 @RouterActivity("second")

```java
@RouterActivity("second")
public class SecondActivity extends Activity {
    @RouterField("uid")
    private int uid;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_second);
        Router.inject(this);
        Log.e("uid", String.valueOf(uid));
    }
}
```

#### 界面跳转

存在以下两种方式跳转到指定界面：

```java
// 方式一
RouterHelper.getSecondActivityHelper().withUid(24).start(this);
// 方式二
Router.startActivity(context, "test://second?uid=233");
```

> 实现机制

#### 定义 Annotation

修饰 class

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface RouterActivity {
    String[] value();
}
```

修饰具体 field

```java
@Inherited
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RouterField {
    String[] value();
}
```

#### 注解处理

文章开始的时候知道本项目是利用 APT 技术生成辅助代码的，为了能更好的全局去分析一下，先看一下 APT 下面生成的代码是什么。

![](http://ww1.sinaimg.cn/large/b10d1ea5ly1feu3fg95v2j21f20y6k3p.jpg)

好吧，其实看着一页截图大概就明白怎么回事了，程序整个流程应该是初始化某个类，其中会涉及到 init() 这个方法，进而通过上述这个方法进行路由和 Activity 的映射存储。具体的实例，就看一下注解处理器如何生成截图中的这个 AptRouterInitializer 这个类吧，看下面这张截图：

![](http://ww1.sinaimg.cn/large/b10d1ea5ly1feu3qdndw9j21by0y64bt.jpg)

核心代码如下所示,做了部分处理：

```java
        // 生成 AptRouterInitializer 这个类
        Set<? extends Element> elements = roundEnv.getElementsAnnotatedWith(RouterActivity.class);
        ClassName activityRouteTableInitializer = ClassName.get("com.thejoyrun.router", "RouterInitializer");
        TypeSpec.Builder typeSpec = TypeSpec.classBuilder((targetModuleName.length() == 0 ? "Apt" : targetModuleName) + "RouterInitializer")
                .addSuperinterface(activityRouteTableInitializer)
                .addModifiers(Modifier.PUBLIC)
                .addStaticBlock(CodeBlock.of(String.format("Router.register(new %sRouterInitializer());", (targetModuleName.length() == 0 ? "Apt" : targetModuleName))));

        // Method build，生成 AptRouterInitializer 类中的 init() 方法
        TypeElement activityRouteTableInitializertypeElement = elementUtils.getTypeElement(activityRouteTableInitializer.toString());
        List<? extends Element> members = elementUtils.getAllMembers(activityRouteTableInitializertypeElement);
        MethodSpec.Builder bindViewMethodSpecBuilder = null;
        for (Element element : members) {
            if ("init".equals(element.getSimpleName().toString())) {
                bindViewMethodSpecBuilder = MethodSpec.overriding((ExecutableElement) element);
                break;
            }
        }
        if (bindViewMethodSpecBuilder == null) {
            return false;
        }
        //  往 init 方法中写入 args.put(xx,yy) 代码
        ClassName activityHelperClassName = ClassName.get("com.thejoyrun.router", "ActivityHelper");
        List<MethodSpec> methodSpecs = new ArrayList<>();
        for (Element element : elements) {
            RouterActivity routerActivity = element.getAnnotation(RouterActivity.class);
            TypeElement typeElement = (TypeElement) element;
            for (String key : routerActivity.value()) {
                bindViewMethodSpecBuilder.addStatement("arg0.put($S, $T.class)", key, typeElement.asType());
            }
            ClassName className = buildActivityHelper(routerActivity.value()[0], activityHelperClassName, (TypeElement) element);

            MethodSpec methodSpec = MethodSpec.methodBuilder("get" + className.simpleName())
                    .addStatement("return new $T()", className)
                    .returns(className)
                    .addModifiers(Modifier.PUBLIC, Modifier.STATIC)
                    .build();
            methodSpecs.add(methodSpec);
        }
        TypeSpec typeSpecRouterHelper = TypeSpec.classBuilder(targetModuleName + "RouterHelper")
                .addModifiers(Modifier.PUBLIC)
                .addMethods(methodSpecs)
                .build();
        // Build Java File
    }
```

主要就像上面介绍的这样了，借助于 JavaPoet 这个代码生成的类库，来根据获取的核心元素生成指定格式的代码。

> 逆向分析

看一下 SecondActivity 的结构,具体代码如下所示：

```java
@RouterActivity({"second", "other2://www.thejoyrun.com/second", "test://www.thejoyrun.com/second"})
public class SecondActivity extends BaseActivity {
    @RouterField("uid")
    private int uid;
    @RouterField("age")
    private int age;
    @RouterField("name")
    private String name;
    @RouterField("man")
    private Boolean man = true;
    @RouterField("manger")
    private Boolean manger;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_second);
        long time = System.currentTimeMillis();
        // 注意这里
        Router.inject(this);
        StringBuilder builder = new StringBuilder();
        builder.append("uid:" + String.valueOf(uid)).append('\n');
        builder.append("age:" + String.valueOf(age)).append('\n');
        builder.append("name:" + String.valueOf(name)).append('\n');
        builder.append("man:" + String.valueOf(man)).append('\n');
        builder.append("manger:" + String.valueOf(manger)).append('\n');
        builder.append("formActivity:" + String.valueOf(formActivity)).append('\n');
        TextView textView = (TextView) findViewById(R.id.text);
        textView.setText(builder.toString());
    }
}
```

在第一行 @RouterActivity 主要作用是将三个路由机制映射到 SecondActivity 上。

在 MyApplication 中：

```java
Router.init("test");
```

在 Router 中找到 init 方法

```java
public static void init(String scheme) {
        Router.sScheme = scheme;
        try {
      Class.forName("com.thejoyrun.router.AptRouterInitializer");
        } catch (ClassNotFoundException e) {
//            e.printStackTrace();
        }
    }
```

通过 forName 找到 AptRouterInitializer 类，其实这个类就是通过 APT 生成的。

```java
public class AptRouterInitializer implements RouterInitializer {
  static {
    // 初始化一个 map
    Router.register(new AptRouterInitializer());}
  @Override
  public void init(Map<String, Class<? extends Activity>> arg0) {
    arg0.put("second", SecondActivity.class);
    arg0.put("other2://www.thejoyrun.com/second", SecondActivity.class);
    arg0.put("test://www.thejoyrun.com/second", SecondActivity.class);
    arg0.put("third", ThirdActivity.class);
  }
}
```

如上所示代码，用一个 static 代码块来初始话一个 Map ，而后进行路由和 Activity 之间的映射存储。接下来看看 Router.inject(this); 

```java
 public static void inject(Activity activity) {
        SafeBundle bundle = new SafeBundle(activity.getIntent().getExtras(), activity.getIntent().getData());
        Class clazz = activity.getClass();
        List<Field> fields = getDeclaredFields(clazz);
        System.out.println(fields.size());
        for (Field field : fields) {
            RouterField annotation = field.getAnnotation(RouterField.class);
            if (annotation == null) {
                continue;
            }
            String type = field.getGenericType().toString();
            field.setAccessible(true);
            String[] names = annotation.value();
            try {
                for (String name : names) {
                    if (!bundle.containsKey(name)) {
                        continue;
                    }
                    if (type.equals("double")) {
                        field.set(activity, bundle.getDouble(name, field.getDouble(activity)));
                        continue;
                    } else if (type.equals("float")) {
                        field.set(activity, bundle.getFloat(name, field.getFloat(activity)));
                        continue;
                    } ...
                    Object defaultValue = field.get(activity);
                    if (field.getGenericType() == String.class) {
                        field.set(activity, bundle.getString(name, (String) defaultValue));
                    } ...
                }
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            }
        }
    }
```

从上面可以看见，其作用是利用反射机制给字段赋予数值。接下来看一下如何去处理跳转过程的：

```jav
 Router.startActivity(this, "test://www.thejoyrun.com/second?uid=233&age=24");
```

进入 Router 中的对应方法看一下：

```java
public static boolean startActivity(Context context, String url) {
        // 是否有拦截机制
  		if (sFilter != null) {
            url = sFilter.doFilter(url);
            if (sFilter.start(context, url)) {
                return true;
            }
        }
        if (TextUtils.isEmpty(url)) {
            return false;
        }
        Uri uri = Uri.parse(url);
  		// 解析 uri 获取到指定的 Activity
        Class clazz = getActivityClass(url, uri);
        if (clazz != null) {
            Intent intent = new Intent(context, clazz);
            intent.setData(uri);
            if (!(context instanceof Activity)) {
                intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            }
          	// 跳转
            context.startActivity(intent);
            return true;
        } else {
            new Throwable(url + "can not startActivity").printStackTrace();
        }
        return false;
    }
```

本质上就是一个 Map 的映射过程，根据路由寻找到 Activity，然后进行跳转机制。但是这段代码中开始一段挺有意思的，一个路由的拦截过程/替换过程。在 Application 全局/手动配置过程中，有以下代码：

```java
// 可选，针对自己的业务做调整
Router.setFilter(new Filter() {
    public String doFilter(String url) {
    	//return url.replace("test://www.thejoyrun.com/","test://");
        return url;
    }
});
```

进行一个 url 的返回(即不做任何操作)或者一个 url 的替换，这样就达到路由替换或者拦截的目的了。验证以下 Filter 中确实提供了这个功能。