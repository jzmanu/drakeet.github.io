LabelView, 继承 TextView, 能够在 TextView 上下左右固定设置文本的 View

![](http://ww3.sinaimg.cn/large/86e2ff85gw1exjirjza6bj212e0o17a9.jpg)

以前，我们如果要写出 <strong><code>我的id: drakeet</code></strong> 这样的条目内容，需要使用两个 <strong><code>TextView</code></strong> 组合，或者一个 <strong><code>TextView</code></strong> 然后每次 <strong><code>setText</code></strong> 的时候，<strong><code>setText ("我的id: " + user.id)</code></strong>, sad...

现在呢，使用 <strong><code>LabelView</code></strong> 吧！
<h4>TODO</h4>
<ol>
	<li>实现上下左右 text 的样式定义（已实现）</li>
	<li>实现上下左右 text 的 margin or padding</li>
</ol>
源代码：<a href="https://github.com/drakeet/LabelView" target="_blank">https://github.com/drakeet/LabelView</a>
