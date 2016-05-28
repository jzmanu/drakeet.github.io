---
title: 浅谈 Android 编程思想和架构
---

我主要是想讲一讲自己对于 接口、模块化、MVP 的一些心得。

有这么一个场景，两个不同的页面，包含了看起来一模一样的界面内容（或者称 frame/UI），这种场景可能很常见，有时看到会说：“哈哈，我可以设计个复用！” 但是遇到一个问题是，<strong>这两个页面需要分别去请求不同的服务端 API，返回下来的数据结构也不一样</strong>（姑且不说去和服务端开发协商），这样就会导致具体的 view holder 或者适配器在绑定数据的时候无法复用，为何说无法复用或难以复用呢？举个例子，比如传进 Apdapter 的 list item 数据内容不一样，你总不能把 item 再拆了，分好几个 list 传进去吧？面向具体编程情况下，适配器得到不同的 items，得对 item 分发数据绑定到 UI，难免要写很多重复的代码。

这时候我们可以采取面向抽象编程，既然<strong>不同的数据对应一样的 UI</strong>，如果它们都实现了一样的接口，这个接口的设计取决于 UI 需要哪些数据块，然后不同的 Item model 去实现这个接口提供数据即可，这时适配器只要持有这个接口类型的 Item 即可，举个通俗的例子：数据模型1和2都实现了 IPost 接口，那么适配器就只要持有 List&lt;IPost&gt; 数据即可，List&lt;数据模型1&gt; 和 List&lt;数据模型2&gt; 都可以视作“一样的鸭子”传递给这个适配器。这样把数据模型的数据块抽取放到了数据模型本身实现，不仅不用写很多重复的分发代码，而且适配器本身都能复用了；当接口需要新的方法，也能驱动着实现者们去实现。
<!--more-->

这就是抽象编程或者接口的好处，接口可以让不同的模型通过同样的方法提供内容，这样它们就可以一定程度上视为同类，有句话说的，<em>如果一个动物走起来像鸭子，叫起来也像鸭子，我们就可以把它当作是鸭子</em>。

同样地，对于有相同、可复用的 UI 这个场景，我们可以更进一步去做，即把 View Holder 化为一个自定义 ViewGroup，或者包裹之。这样做的好处是，更多逻辑不同的地方也都能更好地去复用，ViewHolder 本身出了 Adapter 范围可能难以再复用，而且对于 Adapter，你不需要再纠结 Item UI 的内容点击事件 是要回调到 <strong>Adapter 外</strong>还是在 <strong>Adapter 内</strong>直接绑定数据响应。前者处理方式会导致 Adapter 得暴露很多接口，传递很多数据，经常要从数据集合中重新根据位置取绑定的数据，等等。后者，则会导致 Adapter 变得臃肿，处理过多不应该属于它的业务，而且同样存在着数据重复绑定问题。

这时候如果把 View Holder 化为一个自定义 ViewGroup，那么交互数据的时候就可以进行 UI 数据绑定和响应数据绑定，而监听器建议是在初始化的时候就进行设置，避免 View 被复用的时候，更新数据的同时重复 new 出无谓的监听器对象。

其中，交互给 ViewGroup 这个对象的数据模型也最好使用接口模型，比如同样接收 IPost 接口对象作为数据，这样凡是实现了同样的提供数据接口的类，都能传入这个 ViewGroup 中供其使用。这里提供一个我的常用写法，使用 ViewGroup 关联一个 item 布局：

```java
public class PostView extends LinearLayout implements View.OnClickListener {

    private TextView mSummary;
    private ImageView mAvatar;
    private TextView mUsername;
    private TextView mCreateAt;

    protected IPost mData;


    public PostView(Context context) {
        this(context, null);
    }


    public PostView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }


    public PostView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        inflate(context, R.layout.view_post, this);
    }


    @Override protected void onFinishInflate() {
        super.onFinishInflate();
        mAvatar = (ImageView) findViewById(R.id.avatar);
        mUsername = (TextView) findViewById(R.id.username);
        mCreateAt = (TextView) findViewById(R.id.create_at);
        mSummary = (TextView) findViewById(R.id.summary);
        setOnClickListener(this);
    }


    public void setPost(IPost data) {
        mData = data;
        Glides.loadCircleImage(getContext(), data.getAvatar(), mAvatar);
        mUsername.setText(data.getUsername());
        mCreateAt.setText(data.getCreatedAtFromNow());
        mSummary.setText(data.getSummary());
    }


    @Override public void onClick(View v) {
        Intent intent = StreamActivity.newIntent(getContext(),
                mData.getAvatar(), mData.getUsername(), mData.getUserId(),
                mData.isCurrentUser());
        getContext().startActivity(intent);
    }
}
```

解释了接口的一些实际使用场景之后，我们接下来就很好谈 MVP 架构了，因为可能很多人会在使用 MVP 的时候产生困恼，为什么要搞那么多接口再实现，何不直接调用具体对象的方法？前面讲了那么多，现在应该有点清楚了吧。
<h6><img class="aligncenter" src="http://ww4.sinaimg.cn/large/86e2ff85gw1f1t18zhwsfj20ii0bw0t3.jpg" alt="" width="235" height="151" /></h6>
今年 Android 开发的技术趋势，我觉得一是 RxJava 会继续被更多人接受进而开始使用，二是谷歌花了不少心思的 Data Binding 很可能会迎来正式版，data binding 是实现 MVVM 架构的重要组成部分，介于它还不够完善而且目前还无法提供双向绑定，目前很多人包括俺都还只能停留在个人项目玩一玩的阶段，所以我也是比较青睐于 MVP 架构。

MVP 逐渐流行起来了，必然是有一些好处，不然谁会去管这么一个新兴东西。首先它更加易于测试，Android 平台默认的应用架构导致单元测试变得异常的困难，和 SDK 纠缠在一起的代码，使你无法改变要测试单元的预备状态，无法进行单元测试的准备步骤，而且很多情况，你无法获得测试内容的结果或者状态，也就无法完成断言内容。而使用 MVP 的好处就是能够更易于做测试，同时也能够有更好的复用性和松耦合性。关于 MVP 的 mock &amp; test，谷歌给出了非常好的示例：<a href="https://github.com/googlesamples/android-architecture/tree/todo-mvp/todoapp/app/src" target="_blank">https://github.com/googlesamples/android-architecture/tree/todo-mvp/todoapp/app/src</a>

MVP 即 Model - Presenter - View，各部分之间的通讯，都是双向的，Presenter 持有 View 和 Model 的抽象引用，作为中间人，处理业务逻辑，Model 角色用于调取数据，而 View 则用于展现和控制 UI，它们都由中间人 P 调度。抽象的好处前面说了，如果 M 或 V 有改变，只要换一个实现者就好了，对于 P，可以继续把它们当成提供固定可调用方法对象。

对于包的结构，如果项目比较小，可以把不同的 Presenter 置于同一个 package 下，而如果页面数很多项目模块很多，则可以每一个模块分一套 MVP packages/modules.

现在我们只要在 Activity 或 Fragment 中的生命周期简单做一些 UI 组件初始化等布置，然后一些业务逻辑工作就请求 Presenter 去完成，Presenter 内部持有 View 和 Model，Activity 是 View（MVP 中的 View） 的实现，于是 Presenter 调用 Model 去请求得到数据库或网络数据，完成之后再调用 View 改变 UI 的方法，或者把数据交给 View 去展现。如此，每个角色的代码都会变得很简洁、明确。而且，将业务逻辑放到 Presenter 中去，可以方便去<strong>驱动</strong>避免 Activity 退出而后台线程仍然引用着 Activity，致使资源无法被系统回收从而引起内存泄露。

对于 Presenter 的设计，或者说具体<strong>应该把哪些内容放到 Presenter 中</strong>，是一个关键。Model 并不是必须有的，Model 属于仓库层，如果使用 RxJava 和 Retrofit，可以很清晰地获取数据库内容和网络数据，则可以把 Model 的工作纳入到 Presenter 中。如果带有 Model，则 Presenter 要实现 Model 的回调，在回调中把数据传给 View 或响应。所以 Presenter 必须得有 View 的引用，但可以不必持有 Model.

一个 Activity 可以有多个 Presenter，要用到什么业务就加入什么 Presenter，并且实现这个 Presenter 所需要的 View 接口即可，这就是简单的复用逻辑。

<strong>最后，介绍一种我的重构技巧。</strong>对于原本不是 MVP 的项目，结合 Android Studio 进行重构也很容易，AS 有个好用的功能快捷键是 option + enter，可以用来自动解决错误，利用它，我们可以很方便的把原本都写到 Activity 中的业务抽出，并且不用手动去创建各种接口中的方法和实现方法，步骤大概是这样的：

首先建一个业务的 Presenter 和一个 View 接口，Presenter 中加入 View 接口变量，并写个构造方法用于 View 初始化。而 View 接口，只要先放空即可。然后回到 Activity，implements View 接口，初始化 Presenter，并把自己交给 Presenter，找到原本业务逻辑的地方，把相关代码剪切，然后输入比如 mPresenter.loadData()，这时候，Presenter 中并没有 loadData 这个方法，你只要对着错误按一下 option + enter 就可以出来 “<em>自动在 Presenter 中创建这个方法” </em>的选项，然后自动创建了之后，再跳过去，粘贴刚才的代码，并且在回调的时候，调用 view.hideProgressBar() 方法，同样的，hideProgressBar 这个方法在 View 接口中并没有，于是 option + enter 自动在 View 接口中创建这个方法。这时候 Activity 就会报错，提示你必须实现 hideProgressBar 这个方法，然后你再对着 Activity 按 option + enter 自动生成需要实现的方法的框体，从而进行实现。这样就完成了整个循环驱动重构，是一条 step by step 很简单的套路。

另外，分享我的 Contract 模板（template），Contract 是一个 MVP 的接口统筹声明接口：
<img src="http://ww1.sinaimg.cn/large/86e2ff85gw1f3zgjn58ofj20o00jgq62.jpg" />

附：
<ul>
 	<li><a href="https://github.com/googlesamples/android-architecture/tree/todo-mvp" target="_blank">Google 官方的 MVP 实践</a> - Google 的实践示例中引入了 Contract 角色，是一个非常棒的设计</li>
 	<li><a href="https://github.com/bboyfeiyu/android-tech-frontier/blob/master/issue-12%2FAndroid%E4%B8%8AMVP%E7%9A%84%E4%BB%8B%E7%BB%8D.md#使用mvp" target="_blank">使用 MVP 和后台任务</a> - 对于屏幕旋转 Activity 重建和内存泄漏等问题，MVP 的解决方案</li>
</ul>
&nbsp;
