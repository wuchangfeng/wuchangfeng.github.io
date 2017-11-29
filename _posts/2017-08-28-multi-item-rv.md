---
layout: post
title: 深入理解 Android 之复杂类型的 RecyclerView
date: 2017-08-28 18:55:56 +0800
categories: 
---

深入Android之复杂类型的Item的RecyclerView。这个知识点个人觉得是Android实际开发中必须要具备的。感觉就是完全面向接口编程的能否用好的能力的体现。Google了一下，发现下面这个BetterAdapter项目非常好，拿过来分析一下。同时，我也强烈推荐你阅读下[AdapterDelegates](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0810/3282.html)这篇博客，提供了一种新的思路来实现复杂类型的RV。

### 原生Adapter

```java
public class RecyclerAdapter extends RecyclerView.Adapter<RecyclerAdapter.ViewHolder> {  

  @Override  
  public ViewHolder onCreateViewHolder(ViewGroup viewGroup, int viewType) {  
      if (TYPE_ITEM == viewType) {  
          View v = LayoutInflater.from(viewGroup.getContext()).inflate(R.layout.item, viewGroup, false);  
          return new ViewHolder(v);  
      } else {  
          View view = LayoutInflater.from(viewGroup.getContext()).inflate(R.layout.item_footer, viewGroup, false);  
          return new ViewHolder(view);  
      }  
  }  
  
  @Override  
  public void onBindViewHolder(ViewHolder viewHolder, int i) {  
         viewHolder.textView.setText(mData.get(i) + i);  
  }  
  
  @Override  
  public int getItemCount() {  
      return mData.size();  
  }  

  @Override  
  public int getItemViewType(int position) {  
    if (position + 1 == getItemCount()) {  
        return TYPE_FOOTER;  
    } else {  
        return TYPE_ITEM;  
    }  
}  
}  
```

* onCreateViewHolder 将布局转换为view，并传递到RV封装好的ViewHolder中，该方法中的第二个参数viewType就是进行Item类别区分的。如果类型很多
  就需要写一些Switch或者if-else之类的语句了

* onBindViewHolder 作用就是将viewholder跟数据绑定在一起的，该方法的两个参数分别是viewholder视图与位置。因为这里没有 type 参数, 所以会根据 ViewHolder 的 Instance-of 判断不同类型, 或者也可以在这些 ViewHolder 的基类中处理 onBind，一个说明例子见下面的实例代码：

  ```java
  @Override
     public void onBindViewHolder(ItemViewHolder holder, int position) {
      Thing thing = things.get(position);
      if (thing == Animal) {
          ((AnimalViewHolder) thing).bind((Animal) thing);
      } else if (thing == Car) {
          ((CarViewHolder) thing).bind((Car) thing);
      }
     }
  ```

  这段代码看起来就很乱了, **Instance-of 的检查**和**强制类型转化**使这段代码非常违背设计模式.

* getItemCount 用于返回需要处理的Item个数

* getItemViewType 用于返回具体的位置的Item类型。默认是0，具体重写时候会根据需要返回不同的数字。

从上述函数和参数看到RV已经大致的为我们提供了一些多类型Item的实现机制，利用这些可以实现一些基本的功能，但是实际开发过程中，Item类型是非常复杂的，需要更好的设计模式来调度和支配。

### 预备知识


本文所用的BetterAdapter看着代码关键字应该涉及到了访问者模式，请你先去看一下[Java设计模式-访问者模式](http://alaric.iteye.com/blog/1942517)和[访问者模式系列](https://quanke.gitbooks.io/design-pattern-java/%E6%93%8D%E4%BD%9C%E5%A4%8D%E6%9D%82%E5%AF%B9%E8%B1%A1%E7%BB%93%E6%9E%84%E2%80%94%E2%80%94%E8%AE%BF%E9%97%AE%E8%80%85%E6%A8%A1%E5%BC%8F%EF%BC%88%E4%B8%80%EF%BC%89.html)这篇文章，做好充分的准备。大致需要知道以下概念：

访问者（Visitor）模式：封装一些作用于某种数据结构中的各元素的操作，它可以在不改变这个数据结构的前提下定义作用于这些元素的新的操作。白话一点说就是元素的执行算法可以随着访问者的改变而改变。模式的基本想法如下：首先我们拥有一个由许多对象构成的对象结构，这些对象的类都拥有一个accept方法用来接受访问者对象；访问者是一个接口，它拥有一个visit方法，这个方法对访问到的对象结构中不同类型的元素作出不同的反应；在对象结构的一次访问过程中，我们遍历整个对象结构，对每一个元素都实施accept方法，在每一个元素的accept方法中回调访问者的visit方法，从而使访问者得以处理对象结构的每一个元素。我们可以针对对象结构设计不同的实在的访问者类来完成不同的操作。

1. 抽象访问者（Visitor）角色：定义接口，声明一个或多个访问操作。 
2. 具体访问者（ConcreteVisitor）角色：实现抽象访问者所声明的接口，也就是抽象访问者所声明的各个访问操作。 
3. 抽象元素（Visitable）角色：声明一个接受操作，接受一个访问者对象作为一个参数。 
4. 具体元素结点（ConcreteElement）角色：实现抽象结点所规定的接受操作。 
5. 数据结构对象（ObjectStructure）角色：可以遍历结构中的所有元素，提供一个接口让访问者对象都可以访问每一个元素。 

### 更好的实现方式

我们的目标是：添加多类型时，尽量避免在Adapter中做出更改，符合设计模式中的对扩展开发，对更改封闭。


首先定义访问者接口，提取出Type类型：

```java
public interface Visitable {
    int type(TypeFactory typeFactory);
}
```

每个数据视图继承访问者接口，拥有返回自己本身特性的能力：

```java
public interface Animal extends Visitable {
    /**
     * case animals have names
     */
    String getName();
}
```

每个数据视图就能返回自己特有的特性以及返回自己的数据类型Type的能力了：

```java
public class Car implements Visitable {
    private final String manufacturer;
    private final int powerInPs;

    public Car(String manufacterer, int powerInPs) {
        this.manufacturer = manufacterer;
        this.powerInPs = powerInPs;
    }

    @Override
    public int type(TypeFactory typeFactory) {
        return typeFactory.type(this);
    }

    public String getManufacturer() {
        return manufacturer;
    }

    public int getPowerInPs() {
        return powerInPs;
    }
}
```

抽象工厂类，包含所有需要的类型和对应的ViewHolder：

```java
public interface TypeFactory {
    int type(Duck duck);

    int type(Mouse mouse);

    int type(Car car);

    AbstractBetterViewHolder createViewHolder(View parent, int type);
}
```

通过不同的Item对应的Layout，返回对应的Type类型以及ViewHolder视图。之所以将onCreateViewHolder也放在TypeFactory中，是由于每个Adapter中可能model和type的对应不同, 比如同一个model M , 在Adapter a1中对应type1, 在Adapter a2中对应type2, 所以 onCreateViewHolder的实现也需要放在TypeFactory中解耦.


```java
public class TypeFactoryForList implements TypeFactory {
    @Override
    public int type(Duck duck) {
        return DuckViewHolder.LAYOUT;
    }

    @Override
    public int type(Mouse mouse) {
        return MouseViewHolder.LAYOUT;
    }

    @Override
    public int type(Car car) {
        return CarViewHolder.LAYOUT;
    }

    @Override
    @SuppressLint("DefaultLocale")
    public AbstractBetterViewHolder createViewHolder(View parent, int type) {
        AbstractBetterViewHolder createdViewHolder;
        switch (type) {
            case CarViewHolder.LAYOUT:
                createdViewHolder = new CarViewHolder(parent);
                break;
            case DuckViewHolder.LAYOUT:
                createdViewHolder = new DuckViewHolder(parent);
                break;
            case MouseViewHolder.LAYOUT:
                createdViewHolder = new MouseViewHolder(parent);
                break;
            default:
                throw TypeNotSupportedException.create(String.format("LayoutType: %d", type));
        }
        return createdViewHolder;
    }
}
```

继承RV自己的ViewHolder，提供绑定机制

```java
public abstract class AbstractBetterViewHolder<T> extends RecyclerView.ViewHolder {
    AbstractBetterViewHolder(View view) {
        super(view);
        ButterKnife.bind(this, view);
    }

    public abstract void bind(T element);
}
```

具体数据视图类型对应的ViewHolder

```java
public class MouseViewHolder extends AbstractBetterViewHolder<Mouse> {

    @LayoutRes
    public static final int LAYOUT = R.layout.viewholder_mouse;

    @BindView(R.id.viewholder_mouse_name)
    TextView name;

    public MouseViewHolder(View itemView) {
        super(itemView);
    }

    @Override
    public void bind(Mouse mouse) {
        name.setText(mouse.getName());
    }
}
```

在Adapter中我们的操作就很清爽啦，并且后面的操作也不需要去动Adapter中的代码了

```java
public class BetterAdapter_Visitor extends RecyclerView.Adapter<AbstractBetterViewHolder> {

    private final List<Visitable> elements;
    private final TypeFactory typeFactory;

    public BetterAdapter_Visitor(List<Visitable> elements, TypeFactory typeFactory) {
        this.elements = elements;
        this.typeFactory = typeFactory;
    }

    @Override
    public AbstractBetterViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        Context context = parent.getContext();
        /**
         * attention: {@link viewType} as resource
         */
        View contactView = LayoutInflater.from(context).inflate(viewType, parent, false);
        // 返回对应的viewholder
        return typeFactory.createViewHolder(contactView, viewType);
    }

    @Override
    public void onBindViewHolder(AbstractBetterViewHolder holder, int position) {
        /**
         * attention: unchecked，视图与数据绑定
         */
        holder.bind(elements.get(position));
    }

    @Override
    public int getItemViewType(int position) {
        return elements.get(position).type(typeFactory);
    }

    @Override
    public int getItemCount() {
        return elements.size();
    }

}
```

使用就像下面这样的，并且可以很自然的体会多，后面的类型如果有变化，只需要去实现对应的bean类以及工厂中增加对应的类型就好了，数据的插入也是很扁平化的：

```java
  protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.bind(this);

        TypeFactory typeFactory = new TypeFactoryForList();
        List<Visitable> elementList = Arrays.asList(
                new Car("Mercedes", 625),
                new Duck(),
                new Duck(),
                new Mouse("Mickey"),
                new Duck(),
                new Duck(),
                new Mouse("Not-Donald"),
                new Duck(),
                new Duck(),
                new Mouse("Mini"),
                new Car("Clownscar", 10)
        );

        recyclerView.setLayoutManager(new LinearLayoutManager(this));
        recyclerView.addItemDecoration(new DividerItemDecoration(this, DividerItemDecoration.VERTICAL));
        // 传入数据列表和生产工厂
        recyclerView.setAdapter(new BetterAdapter_Visitor(
                elementList,
                typeFactory
        ));
    }
```

可以见到对于不同类型的数据，传入的参数类型都是一致的，这就可以称为参数的扁平化。


好了，以上就是对于复杂类型Item的RV的一种设计，个人感觉还是访问者这种设计模式的一种实际应用体现，所以，代码阅读量得与经典设计模式相结合呀！！！