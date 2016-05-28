---
title: Retrofit 2.0 + OkHttp 3.0 配置
---

Retrofit 和 OkHttp 都是伟大的 Square 公司开源的伟大项目。前段时间也是从 Retrofit 1.9 升级到 2.0 beta 4 版本，从 OkHttp 2.+ 版本升级到  3.0.1 版本。这两者在各自的这两个大版本升级中，都改变了不少，使得原本的代码都需要进行一些修改才能使用，我也是稍微摸索了几下，如今大致摸清，把一些基础配置，比如设置 Json 转换器、RxJava 适配器、设置 Debug Log 模式、设置超时、错误重连，以及配置 Access token Interceptor 等等一些内容，分享一下。

<h4>引入依赖：</h4>

```groovy
compile 'com.squareup.retrofit2:retrofit:2.0.0-beta4'
compile 'com.squareup.retrofit2:converter-gson:2.0.0-beta4'
compile 'com.squareup.retrofit2:adapter-rxjava:2.0.0-beta4'
compile 'com.squareup.okhttp3:okhttp:3.0.1'
compile 'com.squareup.okhttp3:logging-interceptor:3.0.1'
```

<h4>OkHttp：</h4>

先说 OkHttp 3.0 的配置，3.0 使用层面上的主要改变是，由原本的 okHttp 对象直接各种 set 进行配置改为 Builder 配置模式，所以原本对应的方法应该到 OkHttpClient.Builder 类对象下寻找。我的一些常用配置如下：

```java
HttpLoggingInterceptor interceptor = new HttpLoggingInterceptor();
interceptor.setLevel(HttpLoggingInterceptor.Level.BODY);
OkHttpClient client = new OkHttpClient.Builder()
        .addInterceptor(interceptor)
        .retryOnConnectionFailure(true)
        .connectTimeout(15, TimeUnit.SECONDS)
        .addNetworkInterceptor(mTokenInterceptor)
        .build();
```

解释：

<ul>
	<li>HttpLoggingInterceptor 是一个拦截器，用于输出网络请求和结果的 Log，可以配置 level 为 BASIC / HEADERS / BODY，都很好理解，对应的是原来 retrofit 的 set log level 方法，现在 retrofit 已经没有这个方法了，所以只能到 OkHttp 这边来配置，并且 BODY 对应原来到 FULL.</li>
	<li>retryOnConnectionFailure 方法为设置出现错误进行重新连接。</li>
	<li>connectTimeout 设置超时时间</li>
	<li>addNetworkInterceptor 让所有网络请求都附上你的拦截器，我这里设置了一个 token 拦截器，就是在所有网络请求的 header 加上 token 参数，下面会稍微讲一下这个内容。</li>
</ul>

让所有网络请求都附上你的 token：

```java
Interceptor mTokenInterceptor = new Interceptor() {
    @Override public Response intercept(Chain chain) throws IOException {
        Request originalRequest = chain.request();
        if (Your.sToken == null || alreadyHasAuthorizationHeader(originalRequest)) {
            return chain.proceed(originalRequest);
        }
        Request authorised = originalRequest.newBuilder()
            .header("Authorization", Your.sToken)
            .build();
        return chain.proceed(authorised);
    }
};
```

解释：

<ul>
	<li>那个 if 判断意思是，如果你的 token 是空的，就是还没有请求到 token，比如对于登陆请求，是没有 token 的，只有等到登陆之后才有 token，这时候就不进行附着上 token。另外，如果你的请求中已经带有验证 header 了，比如你手动设置了一个另外的 token，那么也不需要再附着这一个 token.</li>
	<li>header 的 key 通常是 Authorization，如果你的不是这个，可以修改。</li>
</ul>

如果你需要在遇到诸如 401 Not Authorised 的时候进行刷新 token，可以使用 Authenticator，这是一个专门设计用于当验证出现错误的时候，进行询问获取处理的拦截器：

```java
Authenticator mAuthenticator = new Authenticator() {
    @Override public Request authenticate(Route route, Response response)
            throws IOException {
        Your.sToken = service.refreshToken();
        return response.request().newBuilder()
                       .addHeader("Authorization", Your.sToken)
                       .build();        
    }
}
```

<strong>然后，对于以上的两个拦截器，分别使用 OkHttpClient.Builder 对象的 addNetworkInterceptor(mTokenInterceptor) 和 authenticator(mAuthenticator) 即可。</strong>

<h4>Retrofit：</h4>
对于 Retrofit，我的配置是：

```java
Retrofit retrofit = new Retrofit.Builder()
        .baseUrl(AppConfig.BASE_URL)
        .client(client)
        .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
        .addConverterFactory(GsonConverterFactory.create(gson))
        .build();
service = retrofit.create(YourApi.class);
```

解释：

<ul>
	<li>baseUrl: 原来的 setEndPoint 方法变成了 baseUrl</li>
	<li>client 即上面的 OkHttp3 对象</li>
	<li>addCallAdapterFactory 增加 RxJava 适配器</li>
	<li>addConverterFactory 增加 Gson 转换器</li>
</ul>

参考资料：

<ul>
	<li><a href="https://realm.io/news/droidcon-jake-wharton-simple-http-retrofit-2/" target="_blank">Simple HTTP with Retrofit 2</a></li>
</ul>
