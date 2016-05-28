---
title: 编译时动态配置 Android resValue / BuildConfig / Manifest
---

<em>本文参考并得益于我的朋友 <span class="site-title"><a href="http://linroid.com/" target="_blank">@linroid</a> 的文章：<a href="http://linroid.com/2015/07/24/dynamic-endpoint-when-debug-by-xip-io-and-gradle/" target="_blank">xip.io + gradle 在调试时动态设置服务端地址</a>，他写得很好，于是我将在这里写得更普适、完整一点，感谢与分享！</span></em>

当我在修改 `telegram` 源码的时候，我需要提供 2 个版本，一，我自己的版本，二，分享给大家顺便使用的版本。其实二者唯一的差别就是 App 名称不一样，我自己的版本我想叫 `Tel4Drakeet`，但分享给别人，叫这个名字可能不太合适。

因此，我如果每次手动修改再进行编译，显然很麻烦。于是想到借助 `gradle`，想到林的那篇文章，原本我只知道 `buildConfigField` 和类似友盟多渠道打包那样，修改清单文件内容 `manifestPlaceholders`，但这两种都不是我现在想要的，我要的是能够修改 `res value` 的方式，比如修改 `strings.xml` 文件中的 `AppName` 的值。

在林的文章中正好找到这点，貌似在整个互联网上并不容易找到。所以我要总结记录下来：

<h4><strong>使用 gradle 在编译时动态配置 Android resValue
</strong></h4>
在你的 gradle 内容 buildTypes 或者 productFlavors 下面，如 release 体内写上类似：

```groovy
resValue "string", "AppName", "Tel4Drakeet"
```

意为把名为 AppName 的 string value 值改为 Tel4Drakeet，完整地和多个 buildType 结合可以参考我的 <a href="https://github.com/drakeet/Tel4Drakeet/blob/master/tele4drakeet/build.gradle" target="_blank">Tel4Drakeet</a> 的 gradle 文件。
<h4><strong>使用 gradle 在编译时动态设置 Android BuildConfig
</strong></h4>
在同上的地方写上：

```groovy
buildConfigField "String", "ENDPOINT", "\"http://example.com\""
```

gradle sync 一下后，BuildConfig.ENDPOINT 就会被赋值为 http://example.com 就可以供 Java 代码调用了。
<h4><strong>使用 gradle 在编译时动态设置 Android Manifest</strong></h4>
我们在使用友盟多渠道打包的时候，有这么种方式可以在编译的时候动态修改清单文件中的内容，从而实现不同渠道不同标识值。

你需要在 `AndroidManifest.xml` 中，对友盟的渠道进行如下配置：

```xml
<meta-data
    android:name="UMENG_CHANNEL"
    android:value="${UMENG_CHANNEL_VALUE}"/>
```

然后在你的 product flavor 中写上：

```groovy
manifestPlaceholders = [UMENG_CHANNEL_VALUE: "GooglePlay"]
```

完整 gradle 文件可以参考我的这个项目：<a href="https://github.com/drakeet/Meizhi/blob/master/app%2Fbuild.gradle" target="_blank">妹纸&gank.io</a>. 这样清单文件中原本预设的 ${UMENG_CHANNEL_VALUE} 这个值就会被替换为"GooglePlay"。如果要和 Java 配合调用，就是如上所示，结合访问 meta-data 即可，这里不展开了。

:)
