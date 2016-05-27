前不久开始自学 Swift 并且开始开发 iOS App，发现 iOS 的 `UIButton` 有个特性特别喜欢，就是你只需要设置按钮的背景图片即可自动带有按下去按钮变暗的效果，而 Android 如果要做到一样的效果，通常都需要写一个 XML 文件，写 `selector`，而且设计师也要配合着做两种状态的图，即正常状态的图 + 按钮被按下去的状态图。 所以觉得 iOS 的这个 `UIButton` 这点非常实在（=。=），便简单仿造着做了个 `AndroidUIView`，并且提供了 XML 自定义接口，可以设置按下去蒙版的颜色、透明度、形状，还有圆角：

<img class="alignnone wp-image-853" src="/assets/img/2015-03-28-AndroidUIView-1.png" alt="s1" width="171" height="304" /> <img class="alignnone wp-image-852" src="/assets/img/2015-03-28-AndroidUIView-2.png" alt="s2" width="171" height="304" />

分别是正常状态图和按下去之后的状态图

<a title="drakeet/AndroidUIView" href="https://github.com/drakeet/AndroidUIView" target="_blank">https://github.com/drakeet/AndroidUIView</a>

<h3>使用方法：</h3>
1. Gradle

```groovy
dependencies {
    compile 'me.drakeet.library:androiduiview:1.0.0'
}
```

2. 在布局 XML 文件中根节点添加命名空间：

```xml
xmlns:drakeet="http://schemas.android.com/apk/res-auto"
```

3. 之后就可以愉快地使用了，如下（也可以参考截图）：

```xml
<me.drakeet.library.UIButton
    android:layout_width="64dp"
    android:layout_height="64dp"
    android:layout_margin="16dp"
    drakeet:alpha_pressed="80"
    drakeet:color_pressed="#660303ff"
    android:background="@mipmap/ic_launcher"/>

<me.drakeet.library.UIButton
    android:layout_width="64dp"
    android:layout_height="64dp"
    android:layout_margin="16dp"
    drakeet:shape_type="round"
    android:background="@mipmap/avatar"/>

<me.drakeet.library.UIImageView
    android:layout_width="64dp"
    android:layout_height="64dp"
    android:layout_margin="16dp"
    drakeet:shape_type="round"
    android:src="@mipmap/avatar2"/>
```

<h3>自定义属性：</h3>

- `drakeet:alpha_pressed` [integer 默认:48 0-255] -&gt; Alpha of the cover color when pressed
- `drakeet:color_pressed` [color 默认:#9c000000] -&gt; Color of the cover when pressed
- `drakeet:shape_type` [enum (rectangle, round) 默认:rectangle] -&gt; Rectangle or round of cover shape
- `drakeet:radius` [dimension 默认:2dp] -&gt; Add a radius to the cover

<a title="示例 apk" href="https://github.com/drakeet/AndroidUIView/blob/master/sample/sample-release.apk" target="_blank">Demo apk 下载</a>
