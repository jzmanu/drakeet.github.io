今天把 <a title="贝壳单词下载" href="http://fir.im/seashell" target="_blank">贝壳单词</a> 中英语角的 quick return 效果剥离出来写了个 Demo，讲解使用 RecyclerView 写快速返回菜单，效果如下（分别是贝壳单词和 Demo 的截图）：

<img class="alignnone  wp-image-760" src="/assets/img/2015-04-12-anim-1.gif" alt="贝壳单词英语角" width="171" height="306" /> <img class="alignnone wp-image-761" src="/assets/img/2015-04-12-anim-2.png" alt="device-2015-02-14-143056" width="171" height="304" />

通过这篇文章你将了解到的知识有：
<ul>
	<li>RecyclerView 和其适配器的基本使用</li>
	<li>RV 适配多种 Item View 类型写法</li>
	<li>mRecyclerView.setOnScrollListener()</li>
	<li>nineoldandroids 这个强大 View 操作库的使用</li>
</ul>
<!--more-->
首先要在 layout 布局中新建 RecyclerView (以下称 RV) 布局，需要两个层次，一层 RV 置于底层，一层头部的 quick return View，没难度，不细说了，布局如下：

```xml
<FrameLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    xmlns:android="http://schemas.android.com/apk/res/android">

    <android.support.v7.widget.RecyclerView
        android:id="@+id/list"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

    <include layout="@layout/view_header"/>

</FrameLayout>
```

与这个布局相关的实例化关联代码我也都不再说明了，这些简单琐碎的细节如果不懂可以直接看源码，源码在文章末尾会给出。
<h4>关键点1：给第 0 位 item 设置空白占位 View</h4>
说明：为什么要使用这样的方式，因为像 ListView、RecyclerView 这类控件，如果动态改变它整体位置会很卡很糟糕，因此只能将第 0 位 item 设置为空白占位 View，其之上将覆盖那个能够快速返回复出的 Header View。所以这里需要在一开始把这个空白 view 量出来，并且设定进去：

```java
LinearLayoutManager linearLayoutManager = new LinearLayoutManager(this);
linearLayoutManager.setOrientation(LinearLayoutManager.VERTICAL);
mRecyclerView.setLayoutManager(linearLayoutManager);
mDataList = new ArrayList<>();

View paddingView = new View(this);
AbsListView.LayoutParams params = new AbsListView.LayoutParams(
        AbsListView.LayoutParams.MATCH_PARENT, mFlexibleSpaceOffset
);
paddingView.setLayoutParams(params);
paddingView.setBackgroundColor(Color.WHITE);
mListAdapter = new MyListAdapter(mDataList);
mListAdapter.addHeaderView(paddingView);
mRecyclerView.setAdapter(mListAdapter);
```

以上是给 RV 设置布局管理器，和设置 paddingView, 即我在上面说明的空白占位 View。
<h4>关键点2：给 RV 设置滚动监听器</h4>
说明 RV 自带有不错的滚动监听器设置方法，值得一说的是 onScrolled 方法的 dx、dy 参数，差不多可以说是单位时间内滚动的距离，所以我们如果要获取一次滚动操作的总距离，就要把这一次的全部 dy 叠加起来。官方文档对于这两个参数的解释是：
<table class="jd-tagtable">
<tbody>
<tr>
<th>dx</th>
<td>The amount of horizontal scroll.</td>
</tr>
<tr>
<th>dy</th>
<td>The amount of vertical scroll.</td>
</tr>
</tbody>
</table>

我的理解就是单位时间内滚动的距离（可正可负），其命名 d，也和数学的微分表达一样。

在明白了这两个参数之后，就很好办了，代码如下：

```java
mRecyclerView.setOnScrollListener(
        new RecyclerView.OnScrollListener() {
            boolean isIdle;
            int scrollY;

            @Override
            public void onScrollStateChanged(RecyclerView recyclerView, int newState) {
                super.onScrollStateChanged(recyclerView, newState);
                isIdle = newState == RecyclerView.SCROLL_STATE_IDLE;
                if (isIdle) {
                    scrollY = 0;
                }
            }

            @Override
            public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
                super.onScrolled(recyclerView, dx, dy);
                scrollY += dy;
                // show or hide header view
                if (scrollY > 12) {
                    hideHeader();
                } else {
                    showHeader();
                }
            }
        }
);
```

顶部 View 收起和复出的代码，其实很简单，看了就知道了，这个 ViewPropertyAnimator 工具类就是来自 nineoldandroids.

```java
private void showHeader() {
    if (!mHeaderIsShown) {
        ViewPropertyAnimator.animate(mHeader).cancel();
        ViewPropertyAnimator.animate(mHeader).translationY(0).setDuration(200).start();
        mHeaderIsShown = true;
    }
}

private void hideHeader() {
    if (mHeaderIsShown) {
        ViewPropertyAnimator.animate(mHeader).cancel();
        ViewPropertyAnimator.animate(mHeader).translationY(-mFlexibleSpaceOffset).setDuration(200).start();
        mHeaderIsShown = false;
    }
}
```

<h4>关键点3：Adapter 多类型</h4>
说明：由于第 0 个位置我们已经加入了空白 View 占位了，所以实际上，这个 RV 中是有两种 View 的，一种就是那个空白 View，它将被 quick return 覆盖，一种是从第 1 个位置开始的正常 item view。

所以要重写 getItemViewType 方法：

```java
return position == 0 ? TYPE_HEADER : TYPE_CHILD;
```

然后父类的 onCreateViewHolder 会调用到上面这个方法，判断 type 然后返回相应的 View：

```java
@Override
public MyListAdapter.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
    if (mView != null && viewType == TYPE_HEADER) {
        return new ViewHolder(mView);
    } else {
        View v = LayoutInflater.from(parent.getContext()).inflate(R.layout.item_list, parent, false);
        ViewHolder holder = new ViewHolder(v);
        holder.content = (TextView) v.findViewById(R.id.content);
        return holder;
    }
}

@Override
public void onBindViewHolder(MyListAdapter.ViewHolder holder, int position) {
    if (holder.getItemViewType() == TYPE_HEADER) return;
    if (mView != null) {
        position = position - 1;
    }
    final String string = mList.get(position);
    holder.content.setText(string);
}
```

值得注意的是，我预留了一个 public void addHeaderView(View view) 的方法，用来给外部调用设置这个空白 View，也就是我们一开始使用的那个方法，并且在这个 Adapter 中如果这个 HeaderView 不为空，就是有空白 View，就要在 getItemCount 方法中返回的时候，+1：

```java
@Override
public int getItemCount() {
    return mView != null ? mList.size() + 1 : mList.size();
}
```

至此，基本所有的关键的都讲完了，如果还有细节问题，可以再看看我的源代码：

<a href="https://github.com/drakeet/RecyclerQuickReturnDemo" target="_blank">https://github.com/drakeet/RecyclerQuickReturnDemo</a>

（注意：我使用的是最新的 Android Studio 最新的 Gradle，如果你编译或载入不了，也不用太在意，直接用文本编辑器什么的打开 java 源代码文件阅读就好，不一定非要用 IDE）

Demo apk 下载：

<a href="https://github.com/drakeet/RecyclerQuickReturnDemo/raw/master/app-demo.apk" target="_blank">https://github.com/drakeet/RecyclerQuickReturnDemo/raw/master/app-demo.apk</a>
