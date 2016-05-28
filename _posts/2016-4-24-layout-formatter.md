---
title: 当我们谈 XML 布局文件代码的优雅性
---

当我们谈代码的优雅性，是不可以忽略经常在打交道的 Android `XML` 布局文件的书写。有人会问，`XML` 文件内容有什么优雅不优雅的，不都是随便写吗？嗯，是有很多人，根本就没有意识到或者不关心布局文件内容应该怎么写更好看，更漂亮。我觉得，优雅的 Android `XML` 布局文件的内容，应该做到以下几点：

- 不能有多余的空行；
- 不要保留你注释掉的代码（要回溯我们有 git）；
- 尽量避免 hard code（硬编码）；
- 能复用的资源尽量抽出到对应的 value 文件；
- 尽量消除警告、单词拼写错误；
- 不要使用废弃的关键词，如 `fill_parent`、`dip`；
- <strong>属性条目要有序；</strong>

其中，本文特别想讲的就是最后一条，“属性条目要有序”，属性条目指的是，比如一个 `ImageView` 节点下的 `android:id`, `android:layout_width`, `android:paddingLeft`, `android:src` ... 等等这些。<strong>大部分人在书写 Android XML 布局文件的时候，都是想到一个要设置的属性，就随意在原有的属性们之下一行，再加上一行新属性</strong>，这样导致诸如 `android:id` 有时会在最后一行，有时在一堆属性中间，不仅不利于我们在需要 id 的时候，肉眼查找阅读，也会使得整个 `XML` 文件内容没有规则，很凌乱。

所以我大致是给自己定了一个规则，就是 `style` 必须排第一个，紧接着 `visibility`、`id`，然后是一堆 `layout` 属性，再然后是内边距 `padding` 属性，而一些值设定，比如 `color、text、background、src` ... 我则规定它们必须处于最底下，这样在查看值的时候，只要着眼于最底下就可以找到目标属性和它的设定值了。

我们可以看一份随意书写的 `XML` 文件和按照一定顺序规则书写的 `XML` 文件内容对比：
<h6><img class="aligncenter" src="http://ww1.sinaimg.cn/large/86e2ff85gw1f383wa95tej21ge0m5ai0.jpg" /></h6>

上图中，左边是随意书写、乱序的，右图则是按照一定顺序规则有序的，相比之下，我想应该都会觉得右图看起来舒服许多，而且益处应该也是比较明显的。

为了优雅性，以前我都是手动进行排序，`id` 手动写前头，`text` 手动写在末尾，直到最近，因为与人协作，不得不面对大量遗留下来的左图式的 `XML` 文件，忽然想起可以写一个 Idea/AndroidStudio 插件来自动化格式化这些文件内容。于是下载了 Idea 社区版，查看了几下官方文档和一个小示例，就摸索着写起了，最终完成了本文想要推荐 `LayoutFormatter`.

这是我开发并且完全开源的一个 `Idea/AndroidStudio` 插件，通过 `右键 -> Refactor -> Reformat Layout XML`. 即可一键格式化你的 `XML` 文件，甚至可以一键格式化你整个项目或某个文件夹下的所有 `XML` 文件：

<a href="https://github.com/drakeet/LayoutFormatter" target="_blank">https://github.com/drakeet/LayoutFormatter</a>

最近它的更新：

- 调整了格式化 Action 的位置到 `右键 -> Refactor -> Reformat Layout XML`.；
- 支持 撤销；
- 支持 对非 `XML` 文件进行过滤；
- 支持 默认快捷键；
- 支持 一键对整个项目或某个文件夹下的所有 `XML` 文件进行格式化；
- 支持 自动把 `fill_parent` 转为 `match_parent`，把 `dip` 转成 `dp`.

希望你能喜欢并享受它 ^ ^
