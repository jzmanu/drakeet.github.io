![CrashWoodpecker](/assets/img/2015-09-02-crashwoodpecker.png)
因为我的 Android Studio 在使用的过程中, 经常会出现 App 崩溃了, 而 AS 自带的 `logcat` 并没有搜集到异常信息, 特别烦恼, 怎么折腾怎么重连都不痛快, 特别是对于那种 "启动崩" 的异常, 很难马上捕捉到, 丢失也是经常的.

所以我做了这么一个库, 仿造 Square 检测并展示内存泄露的 LeakCanary. 当开发过程中, 如果有没有处理的异常导致 crash, 使用了 CrashWoodpecker 以后, 便会起一个新的美观页面, 以很友好的方式即时展示异常信息. 更多内容可以参看一下我写在 GitHub 的文档:

#### CrashWoodpecker
An uncaught exception handler library like Square's <a href="https://github.com/square/leakcanary">LeakCanary</a>.
<h4>Getting started</h4>
<strong>NOTE: There is a big bug before VERSION 0.9.5, QAQ thank goodness, it has been fixed in version 0.9.5, please update to 0.9.5+.</strong>

In your build.gradle:

```groovy
dependencies {
  debugCompile 'me.drakeet.library:crashwoodpecker:0.9.7'
  releaseCompile 'me.drakeet.library:crashwoodpecker-do-nothing:0.9.7'
}
```

In your Application class:

```java
public class ExampleApplication extends Application {

  @Override public void onCreate() {
    super.onCreate();
    CrashWoodpecker.fly().to(this);
  }
}
```

And in your AndroidManifest.xml file:

```xml
<application
    android:name=".ExampleApplication" // <--
    ...
    ...>
</application>
```

<strong>That is all!</strong> CrashWoodpecker will automatically show an Activity when your app crash with uncaught exceptions in your debug build.

<a href="http://drakeet.me/wp-content/uploads/2015/09/7BDF054B-21AE-4A66-ACBF-6A51B1A0FA96.jpg"><img class="alignnone wp-image-1086" src="http://drakeet.me/wp-content/uploads/2015/09/7BDF054B-21AE-4A66-ACBF-6A51B1A0FA96.jpg" alt="7BDF054B-21AE-4A66-ACBF-6A51B1A0FA96" width="141" height="63" /></a>

<em>Demo apk download: <a href="https://github.com/drakeet/CrashWoodpecker/releases/download/0.9.6/LittleWood.apk" target="_blank">LittleWood.apk</a></em>

源代码: <a href="https://github.com/drakeet/CrashWoodpecker" target="_blank">https://github.com/drakeet/CrashWoodpecker</a>
