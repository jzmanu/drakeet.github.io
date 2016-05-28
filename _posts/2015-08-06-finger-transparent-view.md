---
title: 径向羽化 FingerTransparentView
---

前不久群里有人问怎么实现「类似 Instagram 径向模糊透露手指底部内容」的效果。因为要实时用算法羽化其实挺麻烦的，效率也不见得好，于是我当即很快提供了一个思路便是通过 Ps 做一张中心透明，边缘羽化的图片，再结合 Android 自定义 `View` 常用的 `Canvas` 与 `Paint` 这两个类的混合模式 `Xfermode`，在白板上挖出一个能容纳这张羽化图片的洞，混合去除，再将羽化图片填充进去即可。

<img  src="/assets/img/2015-08-06-finger.gif" alt="device-2015-08-06-133045-d" width="162" height="288" />

说来抽象，结果便是如 GIF 图所示。正好那天比较闲，想得快，做得也快，一个钟头完全靠着自己就几乎完美实现了。

Maven:

```groovy
dependencies {
    compile 'me.drakeet.fingertransparentview:fingertransparentview:1.0.1'
}
```

不过，一般人可能不会用到这个东西，之所以在这边写出来，觉得它还是挺不错的一个自定义 View 教程，开发者可以看看源码学习下也不错，主要可以学到的内容有：

- 双指缩放 `Bitmap`
- 触摸事件分发与传递
- Android 提供的混合绘图接口 `Xfermode`
- 自定义 `View` 的一些流程

源代码：<a href="https://github.com/drakeet/FingerTransparentView" target="_blank">https://github.com/drakeet/FingerTransparentView</a>


遗留小问题：眼尖的同学可能会发现使劲看可以看得出图片边缘，这主要是 `baseColor` 我偷懒直接使用 `Color.WHITE` 衔接不完美导致的，可以通过调整这个变量获得完美衔接，或者重新 P 一张羽化图片也行。
