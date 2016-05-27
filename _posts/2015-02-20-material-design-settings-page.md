设置页面是 Android 开发 App 几乎必须的一个页面。

<img class="alignright wp-image-791" src="/assets/img/2015-02-20-settings.png" alt="device-2015-02-20-132630" width="260" height="462" />

Google 在发布 Material Design 的一些兼容包的时候，一直没有解决的一个大问题便是设置页面。让人很蛋疼的是，如果你继承 PreferenceActivity 来做设置页面的话，会导致你的这个页面 ActionBar 丢失，完全显示不出来，丑，而且官方貌似一直没有解决，真不知怎么想的。所以一般我们的解决办法就是使用 Activity + Fragment 来保留 ActionBar 又能使用简易的 PreferenceFragment。

但是这里又有一个问题，就是没有 Material Design 化，如果你的 APP 是按照 Material Design 风格设计的，那么经常会有这么一个违和的页面，就是『设置页面』，默认状态下，它仍然是 holo 风格，而且似乎很多人不知道如何改变它，Google 自己的很多官方应用，也都没有将它们的设置页面 Material Design 化，这是俺一直忍不了的，于是研究了一番，自己修改实现了这个设置页面的 Material Design（废话太多了，看 demo 真实效果）：→

<strong>这个页面不管在 Android 5.0 上还是 Android 5.0 以下系统，都能保证几乎一模一样的编排和效果。Material Design.  </strong>

下面就来讲讲如何实现，其实很简单。
<h4>关键点一：编写一个 SettingsFragment extends PreferenceFragment</h4>
说明：这是使用设置页面的常见做法了，怕一些新手不懂，所以略微提一下。这个类最基本的应该如下：

```java
/**
 * A placeholder fragment containing a settings view.
 */
public static class SettingsFragment extends PreferenceFragment {

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        addPreferencesFromResource(R.xml.preferences);
    }
}
```

然后在 Activity 中使用 replaceFragment(R.id.settings_container, mSettingsFragment); 将这个 Fragment 置于 settings_container 这个父容器中。

这个 replaceFragment 方法是我自己编写的一个方法，它的内容如下：

```java
public void replaceFragment(int viewId, android.app.Fragment fragment) {
    FragmentManager fragmentManager = getFragmentManager();
    fragmentManager.beginTransaction().replace(viewId, fragment).commit();
}
```

<h4>关键点二：在 res 文件夹下新建一个 xml 文件夹，并且新建一个 preferences.xml 文件</h4>
说明：preferences.xml 这个文件名是任意的，也就是说你只要是 *.xml 就好，做好后 SettingsFragment 就能引用它快速生成设置页面，并且之后还很容易与每一项的设置绑定监听等等。不多说，其实新手看一下这个文件的内容就能够理解它的意思了，我简化了下，一般是类似这样：

```xml
<PreferenceScreen
    android:layout="@layout/preference_item"
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:title="@string/title_activity_setting">
    <PreferenceCategory
        android:layout="@layout/preference_category_widget"
        android:title="人性化自由选择">

        <CheckBoxPreference
            android:layout="@layout/preference_item"
            android:title="贝壳通知栏单词处于最高位置"
            android:key="@string/notify_priority"
            android:summaryOn="当前为最高位置"
            android:summaryOff="当前为最低位置(默认)"
            android:defaultValue="false"/>
        <Preference
            android:layout="@layout/preference_item"
            android:title="在通知栏显示音标"
            android:summary="当前为不显示"
            android:defaultValue="false"/>
    </PreferenceCategory>

    <PreferenceCategory
        android:layout="@layout/preference_category_widget"
        android:title="感谢有你">
        <Preference
            android:layout="@layout/preference_item"
            android:summary="我的博客：http://drakeet.me"
            android:title="作者：drakeet"/>
    </PreferenceCategory>
</PreferenceScreen>
```

之后大家只要在这个文件写设置项就可以相当于写设置页面的布局和内容了，PreferenceScreen 是整个设置页面，PreferenceCategory 是一个设置目录，Preference 或 CheckBoxPreference 是设置目录下的具体设置项。
<h4>最关键点：android:layout 属性</h4>
你会发现我上面的示例 xml 文件中，每一个条目都有使用一个 android:layout 属性，其实如何修改系统默认设置页面成 Material Design 风格的关键就是在这里，你需要开辟一些 layout 文件，按照 Material Design 的尺寸编排要求，实现 Item 的布局，并且，使用的 id 号，必须是系统的！：

<img class="alignnone  wp-image-794" src="/assets/img/2015-02-20-settings-item.jpg" alt="DF5CEF85-196A-4581-9A5D-8D2DA72CE561" width="300" height="88" />

比如我使用的这个 item 布局，其中的

```xml
<TextView
    android:id="@android:id/title"
    ...
/>
```

必须是使用系统的 id，也就是 @android:id/title，这样才会被关联和加载上内容。
说到这里，其实已经都明了了，其他的一些 id 如下：

- summer：`android:id="@android:id/summary"`
- widget_frame：`android:id="@android:id/widget_frame"`
 widget_frame 用来让系统置放控件进去，如 CheckBox 。所以我设置它 android:layout_gravity="right|center_vertical"
目录名其实也是 title：android:id="@android:id/title"
<h4>至此</h4>
这个教程已经很明了了，哈哈，其实是很简单的一件事，所以说起来反而比较别扭，关键就是你要懂得来修改 android:layout 属性，<strong>更多细节和我已经做好的 Material Design 风格的 item 布局，你可以从我的源代码中获取：</strong>

Demo 源代码下载：<a href="https://github.com/drakeet/MaterialSettingsActivityDemo" target="_blank">https://github.com/drakeet/MaterialSettingsActivityDemo</a>


Demo apk：<a href="https://github.com/drakeet/MaterialSettingsActivityDemo/raw/master/app-demo.apk" target="_blank">https://github.com/drakeet/MaterialSettingsActivityDemo/raw/master/app-demo.apk</a>
