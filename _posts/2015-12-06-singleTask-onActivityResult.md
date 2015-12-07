---
title: 解决 singleTask onActivityResult() 无效的问题
---

在 Android 4.X 系统上，如果你将一个 Activity A 的 launchMode 设置为 singleTask 或 singleInstance ，那么当你在 A 中调用 `startActivityForResult()` 的时候，是不会像你想象的那样，在 `onActivityResult()` 中获得你想要的返回结果的，就像[官方文档](http://developer.android.com/intl/zh-cn/reference/android/app/Activity.html#startActivityForResult(android.content.Intent, int, android.os.Bundle) "startActivityForResult()")说的那样：

    For example, if the activity you are launching uses the singleTask launch mode, 
    it will not run in your task and thus you will immediately receive a cancel result.

我们可以使用一个**中介** Activity B 来解决这个问题，思路如下：

 1. 在 A 中通过 `startActivity()` 调用 B ，向 B 中传递你所需的 Intent I ；

 2. 在 B 中通过 `startActivityForResult()` 调用 I ；

 3. 在 B 中通过 `onActivityResult()` 获取 I 的返回结果，通过 [Otto](http://square.github.io/otto/ "Otto") 向 A 发送结果，最后再 finish B 。

当然作为中介，我们需要对 B 进行一些配置。下面是一段简单的实例代码。

首先我们可以构造一个简单的 Agent 类，用于在 A 和 B 之间传递 Intent ：

{% highlight java %}
public class Agent implements Parcelable {
    private Intent intent;
    private int requestCode;

    // 用于判断是否需要在 AgentActivity 中使用 Intent.setComponentName() ；
    // 通常我们使用 Intent ，
    // 都是类似 Intent intent = new Intent(Context, Class) 这样使用，
    // 所以对于传递给 B 的 Intent ，需要修改对应的 Context ；
    // 对于调用系统服务，比如相机，可设置为 false ；
    // 对于 App 内部的 Intent ，则设置为 true ；
    // 但通常来说如果只是调用 App 内部的 Intent ，
    // 其实也没有必要使用中介，直接使用 Otto 就好了。
    private boolean cls;

    public Agent(Intent intent, int requestCode, boolean cls) {
        this.intent = intent;
        this.requestCode = requestCode;
        this.cls = cls;
    }

    public Intent getIntent() {
        return intent;
    }

    public int getRequestCode() {
        return requestCode;
    }

    public boolean isCls() {
        return cls;
    }
    
    // Parcelable 接口的实现...
} {% endhighlight %}

接着我们创建 AgentActivity ，即上文提到的中介 Activity B ：

{% highlight java %}
public class AgentActivity extends Activity {
    public static final String EXTRA_AGENT = "extra_agent";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        if (getIntent() == null) {
            finish();
            return;
        }

        Agent agent = getIntent().getParcelableExtra(EXTRA_AGENT);
        if (agent == null || agent.getIntent() == null) {
            finish();
            return;
        }
        
        // 判断是否需要替换从 A 传递进来的 Intent 的 Context 。
        Intent intent = agent.getIntent();
        if (agent.isCls()) {
            intent.setComponent(
                new ComponentName(this, intent.getComponent().getClassName())
            );
        }

        // 通过 B 调用 A 中构造的 Intent 。
        startActivityForResult(intent, agent.getRequestCode());
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);

        // BusProvider 是 Otto 单例的实现， AgentEvent 是自定义的 Otto 事件。
        BusProvider.getInstance().post(
            new AgentEvent(requestCode, resultCode, data)
        );

        finish();
    }
} {% endhighlight %}

接下来是 AgentEvent 的实现：

{% highlight java %}
public class AgentEvent {
    private int requestCode;
    private int resultCode;
    private Intent data;

    public AgentEvent(int requestCode, int resultCode, Intent data) {
        this.requestCode = requestCode;
        this.resultCode = resultCode;
        this.data = data;
    }

    public int getRequestCode() {
        return requestCode;
    }

    public int getResultCode() {
        return resultCode;
    }

    public Intent getData() {
        return data;
    }
} {% endhighlight %}

最后我们再在 AndroidManifest.xml 里面添加 AgentActivity 的声明：

{% highlight xml %}
<!-- 将 launchMode 设置为 standard ，同时将 theme 设置为不可见。-->
<activity android:name="YOUR_PACKAGE.AgentActivity"
          android:launchMode="standard"
          android:theme="@android:style/Theme.NoDisplay">
</activity> {% endhighlight %}
    
现在所有的准备工作都做完了，你只需要在 A 中这样使用就 OK ：

{% highlight java %}
public class A extend Activity {

    // 为了保证在整个生命周期内能接收到 Otto 的事件，
    // 选择在 onCreate() 和 onDestroy() 中注册与注销订阅。
    @Override
    public void onCreate() {
        ...
        
        BusProvider.getInstance().register(this);
    }
    
    @Override
    public void onDestroy() {
        ...
        
        BusProvider.getInstance().unregister(this);
    }
    
    // 通过 AgentActivity 调用系统相机的示例。
    private void startAgentToCamera() {
        Intent toCamera = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
        
        Agent agent = new Agent(toCamera, YOUR_REQUEST_CODE, false);
        Intent toAgent = new Intent(getActivity(), AgentActivity.class);
        toAgent.putExtra(AgentActivity.EXTRA_AGENT, agent);
        startActivity(toAgent);
    }
    
    // 响应来自 AgentActivity 的事件。
    @Subscribe
    public void onAgentEvent(AgentActivity.AgentEvent event) {
        int requestCode = event.getRequestCode();
        int resultCode = event.getResultCode();
        Intent data = event.getData();
    
        // 处理你接收到的事件...
    }
} {% endhighlight %}

当然从上面的例子我们也可以看出，使用一个中介 Activity 的功能不止于此，你还可以用来处理一些只有 Activity 才能接收的 Intent Action 等。当然具体怎么使用，就请发挥你自己的想象力啦～ 
