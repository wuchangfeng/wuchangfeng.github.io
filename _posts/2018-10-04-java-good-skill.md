---
layout: post
title: Java 中一些好的编码风格
date: 2018-10-04 22:55:41 +0800
categories: 
---


看到一篇非常好的文章，结合自己工作一段时间的感触，非常有体会。转载过来，对一些格式进行修正，并加上自己的一些心得。给自己警示，也分享给大家。

**本文来自 YQS_Love 的CSDN 博客 ，全文地址请点击：https://blog.csdn.net/YQS_Love/article/details/79432048?utm_source=copy**

#### 1.方法名称的意义要明确 

  在提供方法时，一定要明确方法的用意，不然，他人在看到这个方法时会有歧义，以为用错了。
例如： 我想根据 no 获取大于 no 的所有数据，而如下的方法其实是获取等于 no 的意思，而不是大于 no 的，因此会带来歧义。（我之前就犯过这样的错误 :(）

``` java
// 不友好的定义
public List<Book> getBookListByNo(int no);

// 友好的定义
public List<Book> getBookListByGtNo(int no);

```

注意：Gt 是greater than 的缩写，因此不会带来歧义。在NoSQL数据库，比如MongoDB的条件查询中就是这样命名的，如下：

(>) 大于 ->$gt
(<) 小于 -> $lt
(>=) 大于等于 -> $gte
(<= ) 小于等于 -> $lte

#### 2.可根据参数顺序定义方法名

  在给方法提供参数时，方法的命名最好根据参数列表进行命名，这样调用者通过方法即可看到参数的调用顺序，而不必进入方法内查看参数列表，但此规则仅仅适用于参数较少的方法。

例如：我想通过故事 ID 和用户 ID 去获取单个故事或者列表，给定故事 ID 和用户 ID 都是 long 型。

``` java
// 不友好的定义
public Story getStoryId(Long uid,Long storyId);
public Story getStoryBySidAndUid(Long uid,Long sid);

// 友好的定义
public Story getStoryByUidAndSid(Long uid,Long sid);
```

解析： 如果调用方法时没有注意参数的顺序，很容易导致用错，特别是对于参数类型都一样的参数列表，编译器不会报错，将造成严重的后果。如果程序员们都按照这个规则对短参数列表进行这样的命名，注释都省得写了，错误发生的概率也就小了。


#### 3.不返回给客户端映射数据库的实体类

  在给客户端做接口时，返回的数据字段一定要是一个自定义的类，如果返回的结果直接是映射数据库的类，那么这将是灾难。在客户端需要增加返回字段时，你不得不在重新开一个新的接口。虽然有注解可以忽略掉让 dao 层不识别扩展的字段，但对于客户端变化如此之快的业务，这样做也不是明智之举，也会使得原始类变得臃肿，难以维护。（这个只有做过api接口和客户端对接的业务才会有深深的体会）

#### 4.能用包装类型时坚决不用基本类型

  在定义的 bean 时，基本类型最好使用包装类，而数组最好使用 List，因为 List 可以使用 Java 的一些新特性，用起来更方便，也能给他人提供很好的调用，除此之外，对 List 的操作要比操作数组安全得多，当然，这么做有不好之处，出现空指针的概率变大了，编码时要时刻注意。同时，有时候数据库的查询结果可能是 null，因为自动拆箱，用基本数据类型接收有 NPE 风险。（好处还不止这些）

``` java
// 不友好的定义
public class TestClass{

    privete long id;

    private String name;

    private int age;

    private int[] hobbies;

    // 省略构造方法、getter、setter、toString方法
}
```

``` java
// 友好的定义
public class TestClass{

    privete Long id;

    private String name;

    private Int age;

    private List<Integer> hobbies;

    // 省略构造方法、getter、setter、toString方法
}
```

####  5.杜绝 if else 的层层嵌套

  在编码过程中，**如果能够使用反向条件过滤的逻辑，就先过滤掉，这样可以避免 if else 结构的层层嵌套**，使得代码看起来层次结构非常的清晰易懂，也便于维护。如下代码块，你更喜欢看哪块代码呢？

``` java
// 不友好的代码
public StringBuilder testMethod(HttpServletRequest request, Long commentsId){

    StringBuilder rst;                
    if (commentsId == null || commentsId <= 0) {
        rst = ApiResBuilder.json(ActionStatus.PARAMAS_ERROR.inValue(), "invalid-comments_id");
    } else {
        CommentsBean comment = commentsService.get(commentsId);
        if (comment == null) {
            rst = ApiResBuilder.json(CommunityActionStatus.COMMENTS_NOT_EXISTS);
        } else {
            long uid = (Long) request.getAttribute(APIkey.uid);
            if (commentsService.praise(uid, commentsId, comment.getReply2thread())) {
                rst = ApiResBuilder.json(ActionStatus.NORMAL_RETURNED);
            } else {
                rst = ApiResBuilder.json(ActionStatus.UNKNOWN);
            }
        }
    }        
    return rst;
}
```

``` java
// 友好的代码
public StringBuilder testMethod(HttpServletRequest request, Long commentsId) {

    if (Objects.isNull(commentsId) || commentsId <= 0) {
        return ApiResBuilder.json(ActionStatus.PARAMAS_ERROR.inValue(), "invalid-comments_id");
    }

    CommentsBean comment = commentsService.get(commentsId);
    if (comment == null) {
        return ApiResBuilder.json(CommunityActionStatus.COMMENTS_NOT_EXISTS);
    }

    long uid = (Long) request.getAttribute(APIkey.uid);
    if (commentsService.praise(uid, commentsId, comment.getReply2thread())) {
        return ApiResBuilder.json(ActionStatus.NORMAL_RETURNED);
    } 

    return ApiResBuilder.json(ActionStatus.UNKNOWN);
}
```

####  6.不要为了省事，编写较多的内部类

  在编写代码时，如果你的类需要一些框架或者别人用无惨构造方法去构造对象的时候，尽量不要使用内部类的形式，如果非得要使用内部类，一定要将类定义为静态的，否则会导致这个内部类无法被构造。如果你忘记写这个 static 了（除非你非要这么做），那么将会给他人带来无数的“坑”。


####  7.不要吝啬空行

  编写代码时，可以根据业务逻辑将代码分行，用空行告知代码下一步是做什么的，**在你觉得该留空行的地方的留上空行**，而不是整个方法一行空行都没有，那样的代码真的很糟糕。

####  8.不让代码连火车

  编写代码时，一行代码不要太长，如果太长，可在和合适的地方断行。我使用的工具是 IDEA，我的建议是，代码的长度不要超过 IDEA 设定的两条竖线，多那么一丁点也是可以的。同时，方法的参数也应该在合适的地方断行，方便在查看方法时能够快速找到。 
  如下图，超级长的代码，已经远远的超出IDEA设定的竖线。合适的做法是在 appkey 变量名后断行，这样更加优雅和美观。 


####  9.定义 API 接口时，一个参数占据一行

  定义接口时，最好的做法是一个参数占据一行，不要因为参数短，让几个参数占据一行，这使得在查看接口时，不能很好的识别参数的类型、名称等其他属性。

``` java
// 不友好的定义
@RequestMapping(value = "task/stats/golds", method = RequestMethod.GET, headers = "Accept=application/json")
@ApiOperation(value = "获取任务金币发放量", response = TaskGoldStat.class)
@ResponseBody
public JSONResult getUserMessage(@ApiParam("查询时间") @RequestParam(value = "start_time", required = false) Long startTime,@ApiParam("查询时间") @RequestParam(value = "end_time", required = false) Long endTime) {
    try {
        return JSONResult.okResult(userDataStatService.getTaskGoldFlow(startTime, endTime));
    } catch (Exception e) {
        SLogger.error(e, e);
        return JSONResult.failureResult(e.getMessage());
    }
}
```

``` java
// 友好的定义
@RequestMapping(value = "task/stats/golds", method = RequestMethod.GET, headers = "Accept=application/json")
@ApiOperation(value = "获取任务金币发放量", response = TaskGoldStat.class)
@ResponseBody
public JSONResult getUserMessage(@ApiParam("查询时间") @RequestParam(value = "start_time", required = false) Long startTime,
                                 @ApiParam("查询时间") @RequestParam(value = "end_time", required = false) Long endTime) {
    try {
        return JSONResult.okResult(userDataStatService.getTaskGoldFlow(startTime, endTime));
    } catch (Exception e) {
        SLogger.error(e, e);
        return JSONResult.failureResult(e.getMessage());
    }
}
```

注意： 这里有个问题，就是参数可能过长，导致超出了IDEA设定的竖线，这可能违背了上条规则。我的建议是，接口参数定义时，不管这一行代码多长，都要忽略上条规则，如果这个时候将代码断行，识别这个接口的参数就存在一定的障碍。接口一旦定义了，改变和查看的概率远小于业务层代码，因此，可忽略上条规则。


####  10.定义枚举变量，以及一些常量时，名称使用大写

  看过很多前辈代码，有大写的，有小写的，让人感觉很乱。我的建议是将这些变量定义成大写，借助开发工具，能够快速识别这些变量时说明类型。

####  11.利用 IDEA 的功能对代码按照一定的规则分块

  有时候一个类中，代码可能比较多，业务也比较复杂，在他人查看代码时，如果这些业务方法没有进行分类排版，而是错中复杂的穿插在各行，这样的代码是很糟糕的。IDEA 提供了如下命令：

// region 业务模块名称
// endregion 业务模块名称12

在这个标签内的代码，可以灵活的展开和隐藏。这样，我们在编码时，就可以将相同业务的方法放入到此代码中，那么，在review代码时，就变得轻松多了。


####  12.一定要重载 bean 类的 toString() 方法

  构建 bean 对象时，一定要重载 toString()方法，在方法执行抛出异常时，可以直接调用 类的 toString() 方法打印其属性值，便于排查问题，而不是需要 debug 才能知道类的属性值。


#### 13.Object 的 equals 方法最正确的用法

  Object 的 equals 方法容易抛空指针异常，如果明确的知道某个变量不可能为空指针，应使用常量或确定有值的对象来调用 equals。 例如：

``` java
// 不友好的代码
object.equals("test");
```

``` java
// 友好的代码
"test".equals(object);
```

####  14.使用 private 隐藏不想让外界访问的方法

  现在开发项目都是基于 IDE，IDE 的代码提示功能方便了我们的开发。因此，在平时的开发中，如果某些方法不需要让外部所调用而引起错误，那么应该将其修饰为 private 的，这样可以减少错误调用导致的程序错误。编码时脑袋里不要理所当然的都是public。**对比下，Android 开发时候控件的索引都是用 private 修饰的**


####  15.利用集合运算提高程序的效率以及代码简洁度

  做项目开发时会经常遇到这样的情况，从用户表得到了一批用户 id 集合，根据业务需求需要过滤掉一些不符合业务需求的用户 id，将余下的用户 id 在进行别的也操作。那么，经常看到有同事是通过如下代码来处理的：

``` java
List<Long> uidList = new ArrayList<>();
uidList.add(2122346211L);
uidList.add(2126554121L);
uidList.add(2126562651L);
uidList.add(2265621171L);
uidList.add(4126216721L);

List<Long> removeList = new ArrayList<>();
removeList .add(2265621171L);
removeList .add(4126216721L);

uidList.removeAll(removeList);123456789101112
```

  还有一种方法是通过iterator去移除集合内的对象。基础好点的都知道，在对原集合边遍历边移除是会抛异常的，要想边遍历边移除只能用迭代器（iterator）才能够实现。而我推荐的做法是对两集合求差集，而不是调用集合的removeAll()方法，具体代码如下：

``` java
// 集合差集
List resultList = (ArrayList) CollectionUtils.subtract(listA, listB);

// 集合并集
List resultList = (ArrayList) CollectionUtils.union(listA, listB);

// 集合交集
List resultList = (ArrayList) CollectionUtils.intersection(listA, listB);
```

可能有人会疑惑，将Collection强制转化为list会不会抛异常，其实不必担心，看看源码（源码如下）就是知道，CollectionUtils在做差集时将结果集保存到了ArrayList 中，所以不会出现强制转化异常。

``` java
 public static Collection subtract(Collection a, Collection b) {
        ArrayList list = new ArrayList(a);
        Iterator it = b.iterator();

        while(it.hasNext()) {
            list.remove(it.next());
        }

        return list;
   }
```

细心的人会发现，CollectionUtils.subtract()方法就是采用迭代器移除的方法实现的，那么你还有什么理由自己去写一遍这样的代码呢？


####  16.更优雅的写if的条件

  在编码过程中，经常会对一个对象、集合、字符串判断是否是合法的参数，在没有第三方库的时候，我们的条件表达式可能是以下这样写的：

``` java
// 传统的条件表达式
Object object = null;
if(null == object){
    // TODO
}

String string = "";
if(null == string || string.length() <= 0){
    // TODO
}

List<Long> datas = new ArrayList<>();
if(null == datas || datas.size() <= 0){
    // TODO
}
```

  这么写，并没有什么不妥，只是，我们如果查看这个条件时，可能需要一点时间来解读这个if条件到底要做什么，这给我们解读代码带来了一定的障碍，特别是对于集合类或者字符串的判空时。而我建议的写法如下:

``` java
// 更优雅条件表达式
Object object = null;
if(Objects.isNull(object){
    // TODO
}
```

``` java
String string = "";
if(StringUtils.isBlank(string){
    // TODO
}

List<Long> datas = new ArrayList<>();
if(CollectionUtils.isEmpty(datas){
    // TODO
}
```

这些方法还有反向条件的方法：Objects.nonNull(obj),StringUtils.isNotBlank(obj),CollectionUtils.isNotEmpty(obj),这样的代码更加直观的表达了开发者的用意，且使得代码更加的优雅。


####  17.定义类的成员变量时，Boolean 类型不要以 is 开头

  定义为基本数据类型 boolean isSuccess； 的属性，它的方法也是 isSuccess()， 框架在反向解析的时候， “以为”对应的属性名称是 success，导致属性获取不到，进而抛出异常。


####  18.能不用 else 坚决不用

  话不多说，直接上代码，你更喜欢哪块代码呢？

``` java
// 不优雅的代码
public void testMethod(){
    if(...){
        return obj;
    } else {
        return obj;
    }
}
```

``` java
// 优雅的代码
public void testMethod(){
    if(...){
        return obj;
    }
    return obj;
}
```

####  19.创建类时一定要注明作者信息

  在新建立类时，一定要标明这个类是谁创建的，最好连创建时间日期都带上，这么做的用意在于，在维护时能更好的找到责任人（嘻嘻），当然，作用可不止这些。如下给出我在工作中的类注释头。


``` java
/**
 * @Author: YaoQianShu
 * @Date: 2018/2/28
 * @Time: 15:27
 * Copyright © SSY All Rights Reserved.
 */
@Service
public class TestService {
}
```

####  20.使用 Java8 lambda 简化代码，使代码不仅优雅还高大上

   一段普通的方法与 Java8 lambda语法的示例。

``` java
// 普通的代码
List<Integer> datas = new ArrayList<>();
// 省略初始化
for（Integer num : datas）{
    System.out.println(num);
}
```

``` java
// Java8 lambda
List<Integer> datas = new ArrayList<>();
// 省略初始化
datas.stream().forEach(System.out::println);
```

####  21.行注释不要放在代码之后

  **很多人在写代码是为了方便，在写完代码时，将注释直接写在代码之后，而不换行，当代码本身就比较长时，这样的注释可能就超出屏幕的可见范围，这么做是很不可取的。**

``` java
// 不友好的行注释
if(....){ // 这是注释
}

boolean result = StringUtils.isBlank(varA) || Objects.isNull(varB); // 这是注释注释注释注释注释注释注释
```

``` java
// 友好的行注释

// 这是注释
if(....){ 
}

// 这是注释注释注释注释注释注释注释
boolean result = StringUtils.isBlank(varA) || Objects.isNull(varB); 
```

####  22.接口类中的方法和属性不要加任何修饰符号

这一点经常在老司机代码中看见，IDE 会提示没啥用。

``` java
// 不友好的定义
public interface Test{

    /**
     * This is method describe!
     **/
    public void testMethod();
}
```

``` java
// 友好的定义
public interface Test{

    /**
     * This is method describe!
     **/
    void testMethod();
}
```

####  23 不吝啬“{}”

  **在 if/else/for/while/do 语句中必须使用大括号，即使只有一行代码。**，不要写聪明代码。

``` java
// 不友好的代码
if(...) //TODO

// 友好的代码
if(...){
    // TODO
}
```

####  24. 容易被遗忘的关键字 - switch

  在某些控制语句中，如果能用 switch 解决的，推荐使用 switch，例如条件表达式是枚举常量或者其他可使用 switch 完成的，应都用 switch 来控制，switch 不仅可以使代码简洁，且分支跳转效率要比if else 效率高。同时需要注意，**即使 default 分支什么都不处理，我们也应该写上**。补充，最后一个 case 的 break 一定要写。

``` java
// 不友好的代码
if(var == ONE){
    // TODO
} else if(var == TWO){
    // TODO
} else if(....){
}else{
}
```

``` java
// 友好的代码
switch(var){
    case ONE:
        // TODO
    break;
    case TWO：
        // TODO
    break;
    .......
    default:
    break;
}
```

####  25. 集合初始化时尽量指定集合初始值

  ArrayList 尽量使用 ArrayList( int initialCapacity) 初始化，Map 等其他集合类类似。特别是对于一些明确知道集合大小的情况下最为妥当，可以防止集合扩容而牺牲不必要的空间和时间。

``` java
// 不友好的代码（例子为接收Redis的响应结果,外部传入taskKeyList）
Map<String, Response<String>> responseMap = new HashMap<>();
Pipeline pipeline = jedis.pipelined();

for (String taskKey : taskKeyList) {
    String key = createKey(currentTime, uid, taskKey);
    responseMap.put(taskKey, pipeline.get(key));
}
```

``` java
// 友好的代码（此处对HashMap进行了初始化）
Map<String, Response<String>> responseMap = new HashMap<>(taskKeyList.size());
Pipeline pipeline = jedis.pipelined();

for (String taskKey : taskKeyList) {
    String key = createKey(currentTime, uid, taskKey);
    responseMap.put(taskKey, pipeline.get(key));
}
```

####  26.获取系统时间戳用S ystem. currentTimeMillis()

  获取当前毫秒数 System. currentTimeMillis(); 而不是 new Date(). getTime()，或者 JodaTime 的 new DataTime()。如果想获取更加精确的纳秒级时间值，用 System. nanoTime()。 
  Reason:System. currentTimeMillis()是一个 native 方法，并且被 static 修饰，效率更高，而 new 开辟的对象不仅消耗空间，还浪费时间。


####  27.循环体中的语句要考量try-catch性能。 

  循环体中的语句要考量性能，以下操作尽量移至循环体外处理，如定义对象、变量、 
获取数据库连接，进行不必要的 try-catch 操作（这个 try-catch 是否可以移至循环体外） ，并且，将try-catch移至外部后，代码看起来会更优雅。

``` java
// 不友好的代码
public void tryCatchTest1() {

    int divider = 100;

    List<Integer> list = new ArrayList<>(3);
    list.add(12);
    list.add(13);
    list.add(0);

    List<Double> result = new ArrayList<>(3);

    list.stream().forEach(var -> {
        try {
            result.add((double) (divider / var));
        } catch (ArithmeticException e) {
            e.printStackTrace();
        }
    });

    result.stream().forEach(System.out::println);
}
```

``` java
// 友好的代码
public void tryCatchTest() {

    int divider = 100;

    List<Integer> list = new ArrayList<>(3);
    list.add(12);
    list.add(11);
    list.add(0);

    List<Double> result = new ArrayList<>(3);
    try {
        list.stream().forEach(var -> result.add((double) (divider / var)));
    } catch (ArithmeticException e) {
        e.printStackTrace();
    }

    result.stream().forEach(System.out::println);
}
```

1） 使用 Redis 的有序集合时，一定要注意如果想要实现分页时，Redis 不会将分页的大小减 1，而是从 0 到分页大小的下标取数据，所有如果分页取20，那么最终结果是 21个。因为它是按照数组的下标取值的。如果想要取第二页，那么，如果按照 mysql 的分页规则，那么，redis 的size 需要加上 offset 的值，否则分页将不正确。（以上仅限Redis的集合类）

2）在从数据库中获取带状态的数据时，一定要加上状态。比如，现在要获取一个任务，这个任务的状态类型有下线、删除和有效状态，那么，在获取某个任务时，一定要带状态，否则当有相同数据但状态不同时，就会产生错误。

3） 在使用 Map 时，一定要注意数据类型要一致。 

例如：
Map<String,String> datas = new HashMap<>();
datas.put("10001","haha");
datas.put("10002","oo");

Long testKey = 10001L;
String res = datas.get(testKey);

不要以为 res 的值为“haha”，那你就错了，res 的值为 null.查看源码变可以知道，Map 的方法：V get(Object key); 其实是将 key 键当做对象来处理的，因此，会导致取不到结果，正确的做法是将 Long 类型的 key 转化为 String 型或者存储和取值保存一样的方式，如下：

String res = datas.get(String.valueOf(testKey));1

这样就可以得到正确的结果。注意:

*  自定义对象比较记得重载 hashCode 和 equals 方法。 
  在使用 CollectionUtils 时，一定要小心用户自定义对象类集合的比较，因为这些自定义类如果没有重载 hashCode 和 equals 方法时，将得不到你想要的结果。 

* 编写 MySQL 语句时，对每个字段都加‘`field_name`’。 MySQL环境对sql语句中出现关键字时，如果没有加“` `”的话，会报语法错误，导致项目出现 Server Error，因此，以后项目编码过程中最好都带“` `”，防止不必要的bug，增加不必要的Bug修复操作。

* 小心整形数相除得不到小数部分。 两个整形相除会得不到小数部分，因此需要转化除数。 
例如： 
double res = 10 / 3.0; 
必须将3写成 3.0，否则结果将是 3.0，而不是 3.333

* 基本数据类型的包装类对象之间值的比较，全部都用 equals 方法比较。 
  对于 Integer var=?在-128 至 127 之间的赋值， Integer 对象是在 
IntegerCache. cache 产生，会复用已有对象，这个区间内的 Integer 值可以直接使用==进行 
判断，但是这个区间之外的所有数据，都会在堆上产生，并不会复用已有对象，这是一个大坑， 
推荐使用 equals 方法进行判断。


