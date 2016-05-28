---
title: 一种新的 Header View + RecyclerView 实现方式
---

在 GitHub 开源了一种新的 Header View + RecyclerView 实现方式，使用 CoordinatorLayout 把 header 抽离出 RecyclerView，并做到上下滑联动，这样 Java 层代码就能简单和简洁很多，更便于刷新和响应，也不用写多 item view type 逻辑。

<img class="alignright" src="http://ww2.sinaimg.cn/large/86e2ff85gw1f1hn9xvzq5g20k00zk1ky.gif" alt="" width="124" height="221" />

源代码：

<a href="https://github.com/drakeet/RecyclerViewWithHeaderNewPractice" target="_blank">https://github.com/drakeet/RecyclerViewWithHeaderNewPractice</a>

XML 代码层次是这样的：
<img src="http://ww4.sinaimg.cn/large/86e2ff85gw1f1hmtczm7gj21kw0wbgwn.jpg" alt="" />

说明：
<ul>
	<li>可以使用 <a href="https://github.com/henrytao-me/smooth-app-bar-layout" target="_blank">SmoothAppBarLayout</a> 这个第三方库，它是一个提供顺滑滚动的 AppBarLayout 补足库，这一个层次必须置于 RecyclerView 代码下面，也就是界面上是在 RecyclerView 的上层，这样AppBarLayout 包裹的 header views 才能接收到点击事件（其实根本原因是，RV 对于 Padding 部分也会拦截手势所以不得不这么做）。</li>
	<li>如果想要列表上滑的时候，状态栏跟随着滑动显示出阴影效果，可以在 FrameLayout 外再包裹一层 android.support.design.widget.CollapsingToolbarLayout. 可以实现如图效果（注意状态栏阴影）：
<img src="http://ww1.sinaimg.cn/large/86e2ff85gw1f1hs6xb8wdj216e090aaz.jpg" alt="" /></li>
	<li>Demo 中的 UI 来自著名的 App <a href="http://peach.cool" target="_blank">Peach</a>，为了避免写太多无关的、具有干扰性的 UI 代码，俺使用了两部分截图来替代繁琐的 UI 实现，在此声明和感谢  <a href="http://peach.cool" target="_blank">Peach</a>.</li>
</ul>
