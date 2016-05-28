---
title: 使用 MailOtto 做预加载
---

最近我开源了一个专注懒事件的事件总线 <strong>MailOtto</strong>: <a href="https://github.com/drakeet/MailOtto" target="_blank">https://github.com/drakeet/MailOtto</a> 并写了一个用它来做预加载的实践案例：第一个页面预先为第四个页面发起数据加载请求，等用户进入第四个页面，那加载好的数据才会分发给它，若在数据下来前进入第四个页面，也会等完成的时候自动接收到。

这个数据需要 8 秒，如果进入到第四个页面才开始加载，体验就很不好，就算只要 1 秒，也会有一个文本从无到有闪动的过程。如果在第一个页面停留超过 8 秒，它足够完成全程预加载，进入第四个页面里面就能直接拿到数据，可谓完美预加载。

本文就是来介绍一下这个实践案例。

先上这个实践的源码，它位于 <strong>MailOtto</strong> 下的 sample 模块：<a href="https://github.com/drakeet/MailOtto/tree/master/sample/src/main/java/me/drakeet/mailotto/demo" target="_blank">sample</a>, 演示 apk <a href="https://github.com/drakeet/MailOtto/releases" target="_blank">下载</a>

MainActivity 作为模拟一个用户可能停留比较长时间的页面，这样我们就可以在这个页面背后进行为后续的某个页面内容做预加载。

```java
public void onPreload(final View view) {
    ((TextView)view).append(": loading...");
    mHandler.postDelayed(new InnerRunnable(view), 8 * 1000);
}
```

以上代码模拟了一个耗时 8 * 1000 毫秒的后台任务，为了避免 Runnable 匿名内部类引用 MainActivity.this 造成内存泄漏，我使用了一个 InnerRunnable 静态内部类：

```java
public static class InnerRunnable implements Runnable {

    WeakReference<TextView> textViewPreference;


    public InnerRunnable(View textView) {
        this.textViewPreference = new WeakReference<>((TextView) textView);
    }


    @Override public void run() {
        if (textViewPreference.get() != null) {
            textViewPreference.get().append(": done!");
        }
        Mailbox.getInstance().post(new Mail("A mail from MainActivity", TargetActivity.class));
    }
}
```

在这个静态内部类中，run 方法会在 8s 后被调用，从而将加载下来的数据交给 <strong>Mailbox “邮递员”</strong>。

之后路过两个无谓的 NextActivity：

```java
startActivity(NextActivity.getIntent(this, /*page = */2));
```

这两个页面基本什么都没干，纯粹是为了演示跨页面懒通讯，为什么要演示这个内容呢，因为这是为了体现 <strong>MailOtto</strong> 在<strong>跨页面运输</strong>方面天生比 Intent 方式容易得多，而且它支持懒发送懒接收。

到进入第四个页面，即 <strong>TargetActivity</strong>，我们只要告诉“邮递员”：我在家，送信给我吧！“邮递员”就会自动调用你的 <strong>@OnMailReceived</strong>:

```java
Mailbox.getInstance().atHome(this);
```


```java
@OnMailReceived public void onPreloadDataReady(Mail mail) {
    mTextView.setText(mail.content.toString());
}
```

这时候那加载好的数据才会分发给 TargetActivity，若在数据下来前进入 TargetActivity，也会等完成的时候自动接收到。所以我们不必再进行一堆判断或重新请求。

另外，为了防止单例造成内存泄漏，需要在页面结束的时候应该对 Mailbox 标示 “我已经离开了”：

```java
Mailbox.getInstance().leave(this);
```

到此为止，我的例子就结束了。当然这个库绝不仅限于做预加载，只是今天突然发现它天生地适合做预加载，另外它也可以用于取代 <strong>startActivityForResult</strong>，同时拥有 Otto/EventBus 的功能。
