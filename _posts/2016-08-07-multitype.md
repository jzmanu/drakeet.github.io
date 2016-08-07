---
title: Android 复杂的多类型列表视图新写法：MultiType
---

<h1>MultiType</h1>
Android 复杂的多类型列表视图新写法，清晰、灵活、模块开发、插件化思想

<a href="https://github.com/drakeet/MultiType/blob/master/LICENSE" rel="nofollow"><img src="https://img.shields.io/badge/license-Apache%202.0-blue.svg" alt="License" /></a> <img src="https://img.shields.io/maven-central/v/me.drakeet.multitype/multitype.svg" alt="maven-central" />

中文版 | <a href="https://github.com/drakeet/MultiType" target="_blank" rel="nofollow">英文版</a>(English Version)

GitHub: <a href="https://github.com/drakeet/MultiType" target="_blank" rel="nofollow">https://github.com/drakeet/MultiType</a>

这几天晚上回家开始设计我的 TimeMachine 的消息池系统，并抽取出来开源成一个全新的类库： MultiType! 从前，我们写一个复杂的、多 item view types 的列表视图，经常要做一堆繁琐的工作，而且不小心的话代码还堆积严重：我们需要覆写 <code>RecyclerView.Adapter</code> 的 <code>getItemViewType</code> 方法，并新增一些 type 整形常量，而且 <code>ViewHolder</code> 继承、泛型传递、转型也比较糟糕，毕竟 <code>Adapter</code> 只能接受一个泛型……十分麻烦导致过于复杂的页面经常会使用 ScrollView 来实现，一次性加载，而且失去了复用性。

<strong>一旦我们需要新增一些新的 item view types ，就得去修改 <code>Adapter</code> 旧的代码，步骤繁多，侵入较强</strong>。

现在好了，只要三步，不需要修改旧代码，只要无脑往池子里插入新的 type ，会自动连接、分发数据和事件，新增再多的 item types 都能轻松搞定，支持 RV 、复用，代码模块开发，清晰而灵活。若要说为什么这么灵活？ 因为它本来就是为 IM 视图开发的，想想 IM 的消息类型可能有多少种而且新增频繁。
<!--more-->
<h2>接入</h2>
在你的 <code>build.gradle</code>:
```groovy
dependencies {
    compile 'me.drakeet.multitype:multitype:1.1'
}
```
<h2>使用</h2>
<h4>Step 1. 创建一个 class <strong>implements</strong> <code>ItemContent</code>，它将是你的数据类型或 <code>Java bean</code>，示例：</h4>
```java
public class TextItemContent implements ItemContent, Savable {

    @NonNull public String text;

    public TextItemContent(@NonNull String text) {
        this.text = text;
    }

    public TextItemContent(@NonNull byte[] data) {
        init(data);
    }

    @Override public void init(@NonNull byte[] data) {
        String json = new String(data);
        this.text = new Gson().fromJson(json, TextItemContent.class).text;
    }

    @NonNull @Override public byte[] toBytes() {
        return new Gson().toJson(this).getBytes();
    }
}
```
<h4>Step 2. 创建一个 class 继承 <code>ItemViewProvider&lt;C extends ItemContent, V extends ViewHolder&gt;</code>，示例：</h4>
```java
public class TextItemViewProvider
    extends ItemViewProvider<TextItemContent, TextItemViewProvider.TextHolder> {

    static class TextHolder extends RecyclerView.ViewHolder {
        @NonNull final TextView text;

        TextHolder(@NonNull View itemView) {
            super(itemView);
            this.text = (TextView) itemView.findViewById(R.id.text);
        }
    }


    @NonNull @Override
    protected TextHolder onCreateViewHolder(
        @NonNull LayoutInflater inflater, @NonNull ViewGroup parent) {
        View root = inflater.inflate(R.layout.item_text, parent, false);
        TextHolder holder = new TextHolder(root);
        return holder;
    }


    @Override
    protected void onBindViewHolder(
        @NonNull TextHolder holder, @NonNull TextItemContent content, @NonNull TypeItem typeItem) {
        holder.text.setText("hello: " + content.text);
    }
}
```
<h4>Step 3. 好了，你不必再创建新的类文件了，只要往你的 <code>Activity</code> 中加入 <code>RecyclerView</code> 和 <code>List&lt;TypeItem&gt;</code> 即可，示例：</h4>
```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    recyclerView = (RecyclerView) findViewById(R.id.list);

    itemFactory = new TypeItemFactory.Builder().build();
    TypeItem textItem = itemFactory.newItem(new TextItemContent("world"));
    TypeItem imageItem = itemFactory.newItem(new ImageItemContent(R.mipmap.ic_launcher));
    TypeItem richItem = itemFactory.newItem(new RichItemContent("小艾大人赛高", R.mipmap.avatar));

    List<TypeItem> typeItems = new ArrayList<>(80);
    for (int i = 0; i < 20; i++) {
        typeItems.add(textItem);
        typeItems.add(imageItem);
        typeItems.add(richItem);
    }

    /* register the types before setAdapter, that's all right */
    ItemTypePool.register(TextItemContent.class, new TextItemViewProvider());
    ItemTypePool.register(ImageItemContent.class, new ImageItemViewProvider());
    ItemTypePool.register(RichItemContent.class, new RichItemViewProvider());

    recyclerView.setAdapter(new MultiTypeAdapter(typeItems));
}
```
<strong>大功告成！</strong>

你可以阅读源码项目中的 <code>sample</code> 模块获得更多信息和示例，当完整的示例代码运行起来，它是这样子的：

<img src="http://ww2.sinaimg.cn/large/86e2ff85gw1f6hixqglenj21401z4wk9.jpg" width="270" height="486/" />
<h2>Q&amp;A</h2>
<strong>Q: 为什么使用静态或者全局类型池？(Why we need static and single TypePool?)</strong>

A: 我不反对局部或临时类型池的设计，你可以 fork 这个项目自行实现，它们对于内存更加友好（但也只是微小优势而已），但在我看来，全局类型池在多方面更好：
<ul>
 	<li>它能够<strong>显式</strong>连接 Type 和它的 Item View，能够在同一地方统一 register，这将避免分散，带来很好的直观性和可管理性；</li>
 	<li>一个应用不会有超级大量的类型定义，类型 class 和 provider 对象都是非常轻薄的对象，直接静态存于内存，并不会导致内存泄漏和大的内存占用问题，几乎可以忽略；</li>
 	<li>至于要不要支持 optional 的局部类型池参数，我也是不喜欢支持的，前面说了，这是没必要的，而且若是可选（optional）也会使用户疑惑：“到底要还是不要？”</li>
</ul>
因此我喜欢和坚持使用全局静态类型池，它不会带来什么问题，而且好处诸多，有人给我提交了使用反射的方法来自动获取类型连接，为了避免性能话题，我不喜欢反射，而且将类型连接变得复杂和不可见性未必是好事。我一直坚持的原则是：写简单的代码，写可读的代码，实现复杂的需求（你们看我的代码是不是感觉很自然而然而且可读性十分好？）
<h2>题外话</h2>
这个类库成品看来是挺精巧的，或者说轻巧，但一个人从无到有把它设计和创造出来，还是费了很多思考和多次推翻重构，其中有些点看起来可能自然而然，但是它在开发过程中可能都是一个小坎，如果没有找到合适的结构或设计，整体可能就不能搭建起来，使用也可能没那么简单和灵活。所以，要是有人感兴趣，之后可以分享一下开发过程中遇到的问题和思考，还是很有意思的 : )
